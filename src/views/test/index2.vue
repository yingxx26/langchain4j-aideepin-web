<template>
	<div class="container">
		<!-- 输入Markdown内容（实际可从API获取） -->
		<div class="input-area">
      <textarea
				v-model="markdownContent"
				placeholder="粘贴包含Mermaid的Markdown内容..."
				class="markdown-input"
			></textarea>
			<button @click="processContent" class="process-btn">处理并展示</button>
		</div>

		<!-- 处理后的预览 -->
		<div class="preview-area"  >
			<h3>文档预览</h3>
			<div v-html="processedHtml" class="content"></div>
		</div>

		<!-- 导出按钮 -->
		<button
			@click="exportToWord"
			class="export-btn"
			:disabled="!processedHtml"
		>
			导出为Word
		</button>
	</div>
</template>
<script setup>
import { ref, watch, nextTick } from 'vue';
import markdownIt from 'markdown-it';
import mermaid from 'mermaid';
import html2canvas from 'html2canvas';
import axios from 'axios';
import {saveAs} from 'file-saver';

// 原始Markdown内容
const markdownContent = ref(`# 简洁Mermaid渲染示例

\`\`\`mermaid
graph LR
    A[输入Markdown] --> B[markdown-it解析]
    B --> C[提取Mermaid代码]
    C --> D[mermaid渲染SVG]
    D --> E[展示图表]
\`\`\`

\`\`\`mermaid
pie
    title 图表类型占比
    "流程图" : 40
    "时序图" : 30
    "饼图" : 20
    "其他" : 10
\`\`\`
`);

// 处理后的HTML（含图片）
const processedHtml = ref('');
const outputContainer = ref(''); // 容器DOM引用
// 初始化Markdown解析器
const md = markdownIt({
	html: true,
	breaks: true,
	highlight: (str, lang) => lang === 'mermaid'
		? `<pre class="mermaid"><code>${str}</code></pre>`  // 标记mermaid代码块
		: `<pre><code>${str}</code></pre>`
});

// 初始化Mermaid
mermaid.initialize({startOnLoad: false, securityLevel: 'loose'});

/**
 * 处理Markdown：解析为HTML并转换Mermaid为图片
 */
const processContent = async () => {
	if (!markdownContent.value) return;

	// 1. Markdown转原始HTML
	const rawHtml = md.render(markdownContent.value);
// 1. 创建临时DOM元素（不添加到页面，仅用于解析）
	const tempDivaaa = document.createElement('div');
// 2. 将HTML字符串解析为DOM结构
	tempDivaaa.innerHTML = rawHtml;
	await nextTick();
debugger
	// 3. 提取并渲染所有mermaid代码块
	tempDivaaa.querySelectorAll('pre.mermaid').forEach(async (pre, idx) => {
		try {
			const code = pre.textContent.trim();
			const { svg } = await mermaid.render(`m-${Date.now()}-${idx}`, code);
			//pre.outerHTML = `<div class="mermaid-chart">${svg}</div>`;  // 替换为SVG
			// SVG转PNG
			debugger
			const tempDiv = document.createElement('div');
			tempDiv.innerHTML = svg;
			tempDiv.style.position = 'absolute';
			tempDiv.style.left = '-9999px';
			document.body.appendChild(tempDiv);

			const canvas = await html2canvas(tempDiv, {scale: 2});
			const imgSrc = canvas.toDataURL('image/png');
			pre.outerHTML = `<div class="mermaid-img"><img src="${imgSrc}"></div>`;
		} catch (e) {
			pre.outerHTML = `<div class="mermaid-error">渲染失败: ${e.message}</div>`;

		}
	});
	debugger
	processedHtml.value = tempDivaaa.innerHTML;
console.log(processedHtml.value)
};

/**
 * 导出为Word：发送HTML到后端
 */
const exportToWord = async () => {
	try {
		const res = await axios.post(
			'/api/test/export/word ',
			{html: processedHtml.value},
			{responseType: 'blob'}
		);
		saveAs(res.data, `document-${Date.now()}.docx`);
	} catch (e) {
		console.error('导出失败:', e);
		alert('导出失败，请检查Pandoc是否安装');
	}
};
</script>

<style scoped>
.container {
	max-width: 1000px;
	margin: 20px auto;
	padding: 0 20px;
}

.input-area {
	margin-bottom: 20px;
}

.markdown-input {
	width: 100%;
	min-height: 200px;
	padding: 10px;
	border: 1px solid #ddd;
	border-radius: 4px;
}

.process-btn, .export-btn {
	padding: 8px 16px;
	margin-top: 10px;
	background: #2c3e50;
	color: white;
	border: none;
	border-radius: 4px;
	cursor: pointer;
}

.process-btn:hover, .export-btn:hover {
	background: #34495e;
}

.export-btn:disabled {
	background: #ccc;
	cursor: not-allowed;
}

.preview-area {
	border: 1px solid #eee;
	padding: 20px;
	border-radius: 4px;
	margin-bottom: 20px;
}

.content h1, .content h2 {
	color: #2c3e50;
	border-bottom: 1px solid #eee;
}

.mermaid-img {
	margin: 16px 0;
	text-align: center;
}

.mermaid-img img {
	max-width: 100%;
	border: 1px solid #eee;
}

.error {
	color: #e74c3c;
	padding: 10px;
	background: #fef0f0;
	border-radius: 4px;
}
</style>
