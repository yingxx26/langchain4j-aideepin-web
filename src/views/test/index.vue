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
		<div class="preview-area" v-if="processedHtml">
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
import { ref } from 'vue';
import markdownIt from 'markdown-it';
import mermaid from 'mermaid';
import html2canvas from 'html2canvas';
import axios from 'axios';
import { saveAs } from 'file-saver';

// 原始Markdown内容
const markdownContent = ref(`# Pandoc转换示例

## 1. 流程图
\`\`\`mermaid
graph TD
    A["用户"] --> B["前端处理"]
    B --> C["Mermaid转图片"]
    C --> D["生成HTML"]
    D --> E["后端Pandoc转换"]
    E --> F["Word文档"]
\`\`\`

## 2. 说明
- 前端负责Mermaid转图片
- 后端通过Pandoc将HTML转为Word
- 支持复杂格式保留
`);

// 处理后的HTML（含图片）
const processedHtml = ref('');

// 初始化Markdown解析器
const md = markdownIt({ html: true, breaks: true });

// 初始化Mermaid
mermaid.initialize({ startOnLoad: false, securityLevel: 'loose' });

/**
 * 处理Markdown：解析为HTML并转换Mermaid为图片
 */
const processContent = async () => {
	if (!markdownContent.value) return;
debugger
	// 1. Markdown转原始HTML
	const rawHtml = md.render(markdownContent.value);
	let finalHtml = rawHtml;

	// 2. 匹配Mermaid代码块并转换
	const mermaidRegex = /```mermaid([\s\S]*?)```/g;
	const blocks = markdownContent.value.match(mermaidRegex) || [];

	for (const block of blocks) {
		try {
			// 提取Mermaid代码
			const code = block.replace(/```mermaid|\n|```/g, '').trim();

			// 渲染为SVG
			const { svg } = await mermaid.render(`mermaid-${Date.now()}`, code);

			// SVG转PNG
			const tempDiv = document.createElement('div');
			tempDiv.innerHTML = svg;
			tempDiv.style.position = 'absolute';
			tempDiv.style.left = '-9999px';
			document.body.appendChild(tempDiv);

			const canvas = await html2canvas(tempDiv, { scale: 2 });
			const imgSrc = canvas.toDataURL('image/png');

			// 替换代码块为图片
			finalHtml = finalHtml.replace(block, `<div class="mermaid-img"><img src="${imgSrc}"></div>`);
			document.body.removeChild(tempDiv);
		} catch (e) {
			console.error('Mermaid转换失败:', e);
			finalHtml = finalHtml.replace(block, `<div class="error">⚠️ 图表转换失败</div>`);
		}
	}

	processedHtml.value = finalHtml;
};

/**
 * 导出为Word：发送HTML到后端
 */
const exportToWord = async () => {
	try {
		const res = await axios.post(
			'/api/export/word',
			{ html: processedHtml.value },
			{ responseType: 'blob' }
		);
		saveAs(res.data, `document-${Date.now()}.docx`);
	} catch (e) {
		console.error('导出失败:', e);
		alert('导出失败，请检查Pandoc是否安装');
	}
};
</script>

<style scoped>
.container { max-width: 1000px; margin: 20px auto; padding: 0 20px; }
.input-area { margin-bottom: 20px; }
.markdown-input { width: 100%; min-height: 200px; padding: 10px; border: 1px solid #ddd; border-radius: 4px; }
.process-btn, .export-btn { padding: 8px 16px; margin-top: 10px; background: #2c3e50; color: white; border: none; border-radius: 4px; cursor: pointer; }
.process-btn:hover, .export-btn:hover { background: #34495e; }
.export-btn:disabled { background: #ccc; cursor: not-allowed; }
.preview-area { border: 1px solid #eee; padding: 20px; border-radius: 4px; margin-bottom: 20px; }
.content h1, .content h2 { color: #2c3e50; border-bottom: 1px solid #eee; }
.mermaid-img { margin: 16px 0; text-align: center; }
.mermaid-img img { max-width: 100%; border: 1px solid #eee; }
.error { color: #e74c3c; padding: 10px; background: #fef0f0; border-radius: 4px; }
</style>
