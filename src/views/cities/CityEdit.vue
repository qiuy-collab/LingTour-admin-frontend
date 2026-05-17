<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { ArrowDown, ArrowUp, Delete, Plus } from '@element-plus/icons-vue'
import { citiesApi } from '@/api/cities'
import { toI18n, toI18nArray } from '@/types/common'
import I18nInput from '@/components/I18nInput.vue'
import I18nMarkdownEditor from '@/components/I18nMarkdownEditor.vue'
import ImageUpload from '@/components/ImageUpload.vue'
import TemplatePreview from '@/components/TemplatePreview.vue'

const router = useRouter()
const route = useRoute()
const isEdit = computed(() => Boolean(route.params.id))
const loading = ref(false)
const saving = ref(false)

const form = reactive<any>({
  slug: '',
  name: { zh: '', en: '' },
  regionLabel: { zh: '', en: '' },
  adcode: 0,
  heroImage: '',
  heroNarrative: { zh: '', en: '' },
  tags: [],
  editorIntro: { zh: '', en: '' },
  galleryImages: [],
  foodTitle: { zh: '', en: '' },
  foodDescription: { zh: '', en: '' },
  foodImages: [],
  sections: [],
  status: 'draft',
  routeSlugs: [],
})

const newTag = reactive({ zh: '', en: '' })
const newRouteSlug = ref('')

function addTag() {
  if (!newTag.zh.trim() && !newTag.en.trim()) return
  form.tags.push({ zh: newTag.zh.trim(), en: newTag.en.trim() })
  newTag.zh = ''
  newTag.en = ''
}

function removeTag(index: number) {
  form.tags.splice(index, 1)
}

function addRouteSlug() {
  const slug = newRouteSlug.value.trim()
  if (slug && !form.routeSlugs.includes(slug)) {
    form.routeSlugs.push(slug)
    newRouteSlug.value = ''
  }
}

function removeRouteSlug(index: number) {
  form.routeSlugs.splice(index, 1)
}

function addSection() {
  form.sections.push({
    id: `section-${Date.now()}`,
    title: { zh: '', en: '' },
    body: { zh: '', en: '' },
    image: '',
    statLabel: { zh: '', en: '' },
    statValue: { zh: '', en: '' },
    breathImage: '',
    breathQuote: { zh: '', en: '' },
    sortOrder: form.sections.length,
  })
}

function removeSection(index: number) {
  form.sections.splice(index, 1)
  reindexSections()
}

function moveSection(index: number, delta: -1 | 1) {
  const target = index + delta
  if (target < 0 || target >= form.sections.length) return
  ;[form.sections[index], form.sections[target]] = [form.sections[target], form.sections[index]]
  reindexSections()
}

function reindexSections() {
  form.sections.forEach((section: any, index: number) => {
    section.sortOrder = index
  })
}

function fillFromApi(data: any) {
  Object.assign(form, {
    slug: data.slug || '',
    name: toI18n(data.name),
    regionLabel: toI18n(data.regionLabel),
    adcode: data.adcode || 0,
    heroImage: data.heroImage || '',
    heroNarrative: toI18n(data.heroNarrative),
    tags: toI18nArray(data.tags),
    editorIntro: toI18n(data.editorIntro),
    galleryImages: data.galleryImages || [],
    foodTitle: toI18n(data.foodTitle),
    foodDescription: toI18n(data.foodDescription),
    foodImages: data.foodImages || [],
    sections: (data.sections || []).map((section: any, index: number) => ({
      id: section.id || `section-${index}`,
      title: toI18n(section.title),
      body: toI18n(section.body),
      image: section.image || '',
      statLabel: toI18n(section.statLabel),
      statValue: toI18n(section.statValue),
      breathImage: section.breathImage || '',
      breathQuote: toI18n(section.breathQuote),
      sortOrder: section.sortOrder ?? index,
    })),
    status: data.published ? 'published' : 'draft',
    routeSlugs: data.routeSlugs || data.routes?.map((item: any) => item.slug) || [],
  })
}

function toPayload() {
  return {
    slug: form.slug,
    name: form.name,
    regionLabel: form.regionLabel,
    adcode: form.adcode,
    heroImage: form.heroImage,
    heroNarrative: form.heroNarrative,
    tags: form.tags,
    editorIntro: form.editorIntro,
    galleryImages: form.galleryImages,
    foodTitle: form.foodTitle,
    foodDescription: form.foodDescription,
    foodImages: form.foodImages,
    published: form.status === 'published',
    routeSlugs: form.routeSlugs,
    sections: form.sections.map((section: any, index: number) => ({
      title: section.title,
      body: section.body,
      image: section.image,
      statLabel: section.statLabel,
      statValue: section.statValue,
      breathImage: section.breathImage,
      breathQuote: section.breathQuote,
      sortOrder: index,
    })),
  }
}

onMounted(async () => {
  if (!isEdit.value) return
  loading.value = true
  try {
    const res = await citiesApi.getCity(route.params.id as string)
    fillFromApi(res.data.data)
  } catch {
    ElMessage.error('加载城市失败')
    router.push('/admin/cities')
  } finally {
    loading.value = false
  }
})

async function handleSave() {
  saving.value = true
  try {
    if (isEdit.value) {
      await citiesApi.updateCity(route.params.id as string, toPayload())
      ElMessage.success('城市更新成功')
    } else {
      await citiesApi.createCity(toPayload())
      ElMessage.success('城市创建成功')
    }
    router.push('/admin/cities')
  } catch (error: any) {
    ElMessage.error(error?.response?.data?.message || '保存失败')
  } finally {
    saving.value = false
  }
}
</script>

<template>
  <div class="edit-page" v-loading="loading">
    <div class="page-header">
      <h2>{{ isEdit ? '编辑城市' : '新增城市' }}</h2>
      <div>
        <el-button @click="router.push('/admin/cities')">取消</el-button>
        <el-button type="primary" :loading="saving" @click="handleSave">保存</el-button>
      </div>
    </div>

    <div class="editor-shell">
      <el-form class="editor-form" label-position="top">
        <el-card id="section-basic" shadow="never" class="section-card">
          <template #header>基础信息</template>
          <el-row :gutter="16">
            <el-col :span="12">
              <el-form-item label="Slug">
                <el-input v-model="form.slug" placeholder="guangzhou" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="行政区划代码">
                <el-input-number v-model="form.adcode" :min="0" style="width: 100%" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-form-item label="城市名称"><I18nInput v-model="form.name" /></el-form-item>
          <el-form-item label="地区标签"><I18nInput v-model="form.regionLabel" /></el-form-item>
        </el-card>

        <el-card id="section-hero" shadow="never" class="section-card">
          <template #header>头图与叙述</template>
          <el-form-item label="头图"><ImageUpload v-model="form.heroImage" /></el-form-item>
          <el-form-item label="头图叙述"><I18nMarkdownEditor v-model="form.heroNarrative" :rows="6" /></el-form-item>
        </el-card>

        <el-card id="section-editor" shadow="never" class="section-card">
          <template #header>编辑推荐</template>
          <el-form-item label="推荐语"><I18nMarkdownEditor v-model="form.editorIntro" :rows="6" /></el-form-item>
        </el-card>

        <el-card id="section-gallery" shadow="never" class="section-card">
          <template #header>图库与标签</template>
          <el-form-item label="城市图库"><ImageUpload v-model="form.galleryImages" multiple :limit="20" /></el-form-item>
          <el-form-item label="标签">
            <div class="tag-list">
              <el-tag v-for="(tag, index) in form.tags" :key="index" closable @close="removeTag(Number(index))">
                {{ tag.zh || tag.en }} / {{ tag.en || tag.zh }}
              </el-tag>
            </div>
            <div class="inline-row">
              <el-input v-model="newTag.zh" placeholder="中文标签" />
              <el-input v-model="newTag.en" placeholder="English tag" />
              <el-button :icon="Plus" @click="addTag">添加</el-button>
            </div>
          </el-form-item>
        </el-card>

        <el-card id="section-food" shadow="never" class="section-card">
          <template #header>美食模块</template>
          <el-form-item label="美食标题"><I18nInput v-model="form.foodTitle" /></el-form-item>
          <el-form-item label="美食描述"><I18nMarkdownEditor v-model="form.foodDescription" :rows="6" /></el-form-item>
          <el-form-item label="美食图片"><ImageUpload v-model="form.foodImages" multiple :limit="10" /></el-form-item>
        </el-card>

        <el-card shadow="never" class="section-card">
          <template #header>
            <div class="card-header">
              <span>文化段落</span>
              <el-button size="small" type="primary" :icon="Plus" @click="addSection">添加段落</el-button>
            </div>
          </template>
          <div v-for="(section, index) in form.sections" :id="`city-section-${index}`" :key="section.id" class="repeat-item">
            <div class="repeat-header">
              <strong>段落 {{ Number(index) + 1 }}</strong>
              <div>
                <el-button text :icon="ArrowUp" :disabled="Number(index) === 0" @click="moveSection(Number(index), -1)" />
                <el-button text :icon="ArrowDown" :disabled="Number(index) === form.sections.length - 1" @click="moveSection(Number(index), 1)" />
                <el-button text type="danger" :icon="Delete" @click="removeSection(Number(index))" />
              </div>
            </div>
            <el-form-item label="标题"><I18nInput v-model="section.title" /></el-form-item>
            <el-form-item label="正文"><I18nMarkdownEditor v-model="section.body" :rows="6" /></el-form-item>
            <el-form-item label="配图"><ImageUpload v-model="section.image" /></el-form-item>
            <el-row :gutter="12">
              <el-col :span="12"><el-form-item label="数据标签"><I18nInput v-model="section.statLabel" /></el-form-item></el-col>
              <el-col :span="12"><el-form-item label="数据值"><I18nInput v-model="section.statValue" /></el-form-item></el-col>
            </el-row>
            <el-form-item label="呼吸段落图片"><ImageUpload v-model="section.breathImage" /></el-form-item>
            <el-form-item label="呼吸段落引语"><I18nInput v-model="section.breathQuote" type="textarea" :rows="2" /></el-form-item>
          </div>
          <el-empty v-if="form.sections.length === 0" description="暂无段落" :image-size="60" />
        </el-card>

        <el-card shadow="never" class="section-card">
          <template #header>关联与发布</template>
          <el-form-item label="关联路线 Slug">
            <div class="tag-list">
              <el-tag v-for="(slug, index) in form.routeSlugs" :key="slug" type="success" closable @close="removeRouteSlug(Number(index))">
                {{ slug }}
              </el-tag>
            </div>
            <div class="inline-row">
              <el-input v-model="newRouteSlug" placeholder="southern-sea-table" @keyup.enter="addRouteSlug" />
              <el-button :icon="Plus" @click="addRouteSlug">添加</el-button>
            </div>
          </el-form-item>
          <el-form-item label="发布状态">
            <el-radio-group v-model="form.status">
              <el-radio value="draft">草稿</el-radio>
              <el-radio value="published">已发布</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-card>
      </el-form>

      <TemplatePreview type="city" :model="form" />
    </div>
  </div>
</template>

<style scoped>
.edit-page { padding-bottom: 40px; }
.page-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 18px; }
.page-header h2 { margin: 0; font-size: 20px; }
.editor-shell { display: grid; grid-template-columns: minmax(0, 1fr) 390px; gap: 20px; align-items: start; }
.section-card { margin-bottom: 16px; }
.card-header, .repeat-header, .inline-row, .tag-list { display: flex; align-items: center; gap: 8px; }
.card-header, .repeat-header { justify-content: space-between; }
.inline-row { width: 100%; }
.inline-row .el-input { max-width: 260px; }
.tag-list { flex-wrap: wrap; min-height: 28px; margin-bottom: 8px; }
.repeat-item { padding: 14px; margin-bottom: 14px; border: 1px solid #ebeef5; border-radius: 8px; background: #fafbfc; }
@media (max-width: 1100px) { .editor-shell { grid-template-columns: 1fr; } }
</style>
