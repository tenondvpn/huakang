<template>
  <el-card class="markdown-viewer" shadow="hover">
    <template #header>
      <div class="card-header">
        <span>{{ markdownTitle }}</span>
        <el-button type="primary" size="small" @click="refresh">刷新</el-button>
      </div>
    </template>

    <el-scrollbar v-loading="loading" max-height="600px">
      <div class="markdown-content" v-html="renderedMarkdown"></div>
    </el-scrollbar>

    <template #footer v-if="error">
      <el-alert type="error" :closable="false">{{ error }}</el-alert>
    </template>
  </el-card>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import MarkdownIt from 'markdown-it'
import MarkdownItHighlightjs from 'markdown-it-highlightjs'
import 'highlight.js/styles/github.css' // 高亮样式（可换其他主题，如 'atom-one-dark.css'）
import qs from 'qs'
import axios from 'axios'

// 初始化 Markdown 解析器
const md = new MarkdownIt({
  html: true,        // 允许 HTML 输出
  linkify: true,     // 自动转链接
  typographer: true  // 美化排版
}).use(MarkdownItHighlightjs) // 添加语法高亮

const props = defineProps({
    schedule_id: Number,
})

const loading = ref(false)
const error = ref('')
const markdownSource = ref('')
const markdownTitle = ref('')
const renderedMarkdown = computed(() => {
  try {
    return md.render(markdownSource.value)
  } catch (err) {
    error.value = '渲染失败: ' + err.message
    return ''
  }
})

const refresh = () => {
  loading.value = true
  setTimeout(() => {
    loading.value = false
  }, 300) // 模拟刷新动画
}

onMounted(() => {
    getFileContent()
    refresh()
})

const getFileContent = () => {
    axios
        .post('/pipeline/get_log_content/', qs.stringify({
            'schedule_id': props.schedule_id,
            'file_name': "user.md",
            'rerun_id': 0,
        }))
        .then(response => {
            console.log(response.data)
            markdownSource.value = response.data.file_content.replace(/\\n/g, '\n')
            refresh()
        })
        .catch(error => {
            ElMessage.success("获取文件内容失败：" + error);
        })
}

</script>

<style scoped>
.markdown-viewer {
  max-width: 800px;
  margin: 20px auto;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.markdown-content {
  padding: 16px;
  line-height: 1.6;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* 自定义 Markdown 样式（可选，覆盖默认） */
.markdown-content :deep(h1) {
  font-size: 2em;
  border-bottom: 1px solid #eee;
}

.markdown-content :deep(code) {
  background: #f6f8fa;
  padding: 2px 4px;
  border-radius: 3px;
}
</style>