<template>
	<div class="converter-container">
		<!-- Markdown编辑器 -->
		<v-md-editor
			v-model="rawMarkdown"
			height="500px"
			:toolbar="toolbar"
			@save="handleConvert"
		/>
		<!-- 下载按钮 -->
		<button v-if="wordUrl" @click="downloadWord" class="download-btn">
			下载Word文件
		</button>
	</div>
</template>

<script setup>
import { ref } from 'vue';
import VMdEditor from '@kangc/v-md-editor';
import '@kangc/v-md-editor/lib/style/base-editor.css';
import vuepressTheme from '@kangc/v-md-editor/lib/theme/vuepress.js';
import '@kangc/v-md-editor/lib/theme/style/vuepress.css';
import mermaid from 'mermaid';
import axios from 'axios';
import MarkdownIt from 'markdown-it';
import mermaidPlugin from '@kangc/v-md-editor/lib/plugins/mermaid/cdn.js';
// 初始化编辑器（支持Mermaid预览）
VMdEditor.use(vuepressTheme, {
	extend(md) {
		md.use(mermaidPlugin, { mermaid });
	},
});

const rawMarkdown = ref(`# 前端渲染Mermaid示例
\`\`\`mermaid
pie
  title 数据占比
  "A" : 30
  "B" : 70
\`\`\`
`);
const wordUrl = ref('');
const toolbar = ['bold', 'italic', 'code', 'save']; // 自定义工具栏
const mdParser = new MarkdownIt(); // 用于解析Markdown

// 核心：将Mermaid代码渲染为Base64图片并替换
const replaceMermaidWithImage = async (markdown) => {
	// 匹配Mermaid代码块（```mermaid ... ```）
	const mermaidRegex = /```mermaid([\s\S]*?)```/g;
	const matches = markdown.matchAll(mermaidRegex);
	let processedMd = markdown;

	for (const match of matches) {
		const fullBlock = match[0];
		const mermaidCode = match[1].trim();

		try {
			// 初始化Mermaid（设置渲染为SVG，再转为Base64）
			mermaid.initialize({ startOnLoad: false, output: 'svg' });
			// 渲染为SVG字符串
			const { svg } = await mermaid.render('mermaid-chart', mermaidCode);
			// SVG转为Base64图片
			const base64 = `data:image/svg+xml;base64,${btoa(unescape(encodeURIComponent(svg)))}`;
			// 替换原Mermaid代码块为图片标签
			processedMd = processedMd.replace(fullBlock, `<img src="${base64}" alt="mermaid-chart">`);
		} catch (error) {
			console.error('Mermaid渲染失败', error);
		}
	}
	return processedMd;
};

// 提交处理后的Markdown到后端
const handleConvert = async () => {
	try {
		// 1. 前端渲染Mermaid为图片，替换原Markdown
		const processedMd = await replaceMermaidWithImage(rawMarkdown.value);
		// 2. 提交给后端生成Word
		const response = await axios.post(
			'/api/md-to-word',
			{ content: processedMd },
			{ responseType: 'blob' }
		);
		// 3. 生成下载链接
		const blob = new Blob([response.data], { type: 'application/vnd.openxmlformats-officedocument.wordprocessingml.document' });
		wordUrl.value = URL.createObjectURL(blob);
	} catch (error) {
		console.error('转换失败', error);
	}
};

// 下载Word文件
const downloadWord = () => {
	const a = document.createElement('a');
	a.href = wordUrl.value;
	a.download = 'report.docx';
	a.click();
	URL.revokeObjectURL(wordUrl.value);
};
</script>
