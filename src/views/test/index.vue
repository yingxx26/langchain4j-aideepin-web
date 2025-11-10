<template>
	<div>
		<textarea v-model="markdown" placeholder="输入Markdown内容..." rows="10"></textarea>
		<button @click="processAndConvert">转换为Word</button>
		<a v-if="downloadUrl" :href="downloadUrl" download="document.docx">下载Word</a>
	</div>
</template>

<script>
import axios from 'axios';
import mermaid from 'mermaid';
import html2canvas from 'html2canvas';

export default {
	data() {
		return {
			markdown: '',
			processedMarkdown: '',
			downloadUrl: null,
		};
	},
	methods: {
		async processAndConvert() {
			// 处理 Mermaid 代码为 Base64 图片
			const processed = await this.processMarkdownWithMermaid();
			// 发送到后端
			const response = await axios.post('/api/convert', processed, { responseType: 'blob' });
			const url = window.URL.createObjectURL(new Blob([response.data]));
			this.downloadUrl = url;
		},
		async processMarkdownWithMermaid() {
			const lines = this.markdown.split('\n');
			let result = [];
			let currentCodeBlock = '';
			let inMermaid = false;

			for (const line of lines) {
				if (line.startsWith('```mermaid')) {
					inMermaid = true;
					currentCodeBlock = '';
				} else if (line.startsWith('```')) {
					if (inMermaid) {
						const svg = await this.generateMermaidImage(currentCodeBlock);
						const base64 = await this.svgToBase64(svg);
						result.push(`![Mermaid Diagram](data:image/png;base64,${base64})`);
						inMermaid = false;
					} else {
						result.push(line);
					}
				} else if (inMermaid) {
					currentCodeBlock += line + '\n';
				} else {
					result.push(line);
				}
			}

			return result.join('\n');
		},
		async generateMermaidImage(code) {
			mermaid.initialize({ startOnLoad: false });
			const id = 'mermaid-' + Date.now();
			return await mermaid.render(id, code);
		},
		async svgToBase64(svg) {
			const parser = new DOMParser();
			const doc = parser.parseFromString(svg, 'image/svg+xml');
			const svgElement = doc.querySelector('svg');
			const canvas = document.createElement('canvas');
			const ctx = canvas.getContext('2d');

			const width = parseFloat(svgElement.getAttribute('width')) || 400;
			const height = parseFloat(svgElement.getAttribute('height')) || 200;
			canvas.width = width;
			canvas.height = height;

			const image = new Image();
			image.onload = () => {
				ctx.drawImage(image, 0, 0);
			};
			image.src = 'data:image/svg+xml;base64,' + btoa(new XMLSerializer().serializeToString(svgElement));

			return new Promise(resolve => {
				canvas.toDataURL('image/png', (dataUrl) => {
					resolve(dataUrl.split(',')[1]);
				});
			});
		}
	}
};
</script>
