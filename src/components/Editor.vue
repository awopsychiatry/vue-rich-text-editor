<template>
  <div class="flex flex-col gap-8 p-8">
    <div class="flex gap-4">
      <div class="flex-1">
        <div id="editor-toolbar" class="mb-2"></div>
        <div ref="quillContainer" class="h-64 border rounded bg-white"></div>
        <div class="mt-4 flex gap-2">
          <button @click="exportMarkdown" class="btn">Export .md</button>
          <button @click="exportHTML" class="btn">Export .html</button>
          <button @click="exportPDF" class="btn">Export .pdf</button>
        </div>
      </div>
      <div class="flex-1">
        <h3 class="mb-2 font-semibold">Markdown Preview</h3>
        <pre class="p-2 bg-gray-100 border rounded h-64 overflow-auto">{{ markdown }}</pre>
        <h3 class="mt-4 mb-2 font-semibold">HTML Preview</h3>
        <div class="p-2 bg-gray-50 border rounded h-64 overflow-auto" v-html="html"></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Quill from 'quill'
import 'quill/dist/quill.snow.css'
import MarkdownIt from 'markdown-it'
import jsPDF from 'jspdf'

const quillContainer = ref(null)
const quillInstance = ref(null)
const md = new MarkdownIt()

const html = ref('')
const markdown = ref('')

function updatePreviews() {
  html.value = quillInstance.value.root.innerHTML
  markdown.value = md.render(quillInstance.value.root.innerHTML)
}

onMounted(() => {
  const toolbarOptions = [
    [{ header: [1, 2, false] }],
    ['bold', 'italic', 'underline', 'strike'],
    ['blockquote', 'code-block'],
    [{ list: 'ordered' }, { list: 'bullet' }],
    ['link', 'image'],
    ['clean'],
  ]
  quillInstance.value = new Quill(quillContainer.value, {
    theme: 'snow',
    modules: { toolbar: toolbarOptions },
  })
  quillInstance.value.on('text-change', updatePreviews)
  updatePreviews()
})

function exportMarkdown() {
  const blob = new Blob([markdown.value], { type: 'text/markdown' })
  downloadBlob(blob, 'document.md')
}
function exportHTML() {
  const blob = new Blob([html.value], { type: 'text/html' })
  downloadBlob(blob, 'document.html')
}
function exportPDF() {
  const doc = new jsPDF()
  doc.text(html.value, 10, 10)
  doc.save('document.pdf')
}
function downloadBlob(blob, filename) {
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = filename
  link.click()
  URL.revokeObjectURL(link.href)
}
</script>

<style scoped>
.btn {
  @apply px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition;
}
</style>
