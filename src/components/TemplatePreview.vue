<script setup lang="ts">
import { computed, ref } from 'vue'
import { pickI18n } from '@/types/common'

const props = defineProps<{
  type: 'city' | 'route' | 'product'
  model: Record<string, any>
}>()

const locale = ref<'zh' | 'en'>('zh')
const text = (value: unknown, fallback = '待填写') => pickI18n(value, locale.value) || fallback
const image = (value: unknown) => (typeof value === 'string' && value ? value : '')

const title = computed(() => {
  if (props.type === 'city') return text(props.model.name, '城市名称')
  if (props.type === 'route') return text(props.model.title, '路线标题')
  return text(props.model.name, '商品名称')
})

const heroImage = computed(() => {
  if (props.type === 'city') return image(props.model.heroImage)
  if (props.type === 'route') return image(props.model.coverImage)
  return image(props.model.image)
})

function scrollToEditor(id: string) {
  document.getElementById(id)?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}
</script>

<template>
  <aside class="template-preview">
    <div class="preview-toolbar">
      <strong>实时预览</strong>
      <el-segmented v-model="locale" :options="[{ label: '中文', value: 'zh' }, { label: 'EN', value: 'en' }]" size="small" />
    </div>

    <div class="preview-canvas">
      <section class="preview-hero" @click="scrollToEditor('section-basic')">
        <img v-if="heroImage" :src="heroImage" alt="" />
        <div v-else class="image-placeholder">封面图</div>
        <div class="hero-copy">
          <p>{{ type === 'product' ? text(model.tag, '商品标签') : text(model.regionLabel || model.cultureTag, '模板标签') }}</p>
          <h3>{{ title }}</h3>
          <span v-if="type === 'product'">{{ model.currency }} {{ Number(model.price || 0).toFixed(2) }}</span>
        </div>
      </section>

      <template v-if="type === 'city'">
        <section class="preview-block" @click="scrollToEditor('section-hero')">
          <h4>城市叙述</h4>
          <p>{{ text(model.heroNarrative) }}</p>
        </section>
        <section class="preview-block" @click="scrollToEditor('section-editor')">
          <h4>编辑推荐</h4>
          <p>{{ text(model.editorIntro) }}</p>
        </section>
        <section class="preview-grid" @click="scrollToEditor('section-gallery')">
          <div v-for="img in (model.galleryImages || []).slice(0, 4)" :key="img" class="thumb"><img :src="img" alt="" /></div>
          <div v-if="!(model.galleryImages || []).length" class="thumb empty">图库</div>
        </section>
        <section v-for="(section, index) in model.sections || []" :key="section.id || index" class="preview-block" @click="scrollToEditor(`city-section-${index}`)">
          <h4>{{ text(section.title, `段落 ${Number(index) + 1}`) }}</h4>
          <p>{{ text(section.body) }}</p>
        </section>
        <section class="preview-block" @click="scrollToEditor('section-food')">
          <h4>{{ text(model.foodTitle, '美食模块') }}</h4>
          <p>{{ text(model.foodDescription) }}</p>
        </section>
      </template>

      <template v-else-if="type === 'route'">
        <section class="preview-block" @click="scrollToEditor('section-story')">
          <h4>{{ text(model.duration, '路线时长') }} · {{ text(model.audience, '目标人群') }}</h4>
          <p>{{ text(model.summary) }}</p>
        </section>
        <section v-for="(stop, index) in model.stops || []" :key="stop.id || index" class="preview-stop" @click="scrollToEditor(`route-stop-${index}`)">
          <span>{{ stop.time || '--:--' }}</span>
          <div>
            <h4>{{ text(stop.stopName, `站点 ${Number(index) + 1}`) }}</h4>
            <p>{{ text(stop.story) }}</p>
          </div>
        </section>
      </template>

      <template v-else>
        <section class="preview-block" @click="scrollToEditor('section-story')">
          <h4>商品故事</h4>
          <p>{{ text(model.story) }}</p>
        </section>
        <section class="preview-specs" @click="scrollToEditor('section-details')">
          <div><span>材质</span><b>{{ text(model.material) }}</b></div>
          <div><span>尺寸</span><b>{{ text(model.dimensions) }}</b></div>
          <div><span>产地</span><b>{{ text(model.origin) }}</b></div>
        </section>
        <section class="preview-grid" @click="scrollToEditor('section-images')">
          <div v-for="img in (model.gallery || []).slice(0, 4)" :key="img" class="thumb"><img :src="img" alt="" /></div>
          <div v-if="!(model.gallery || []).length" class="thumb empty">图库</div>
        </section>
      </template>
    </div>
  </aside>
</template>

<style scoped>
.template-preview {
  position: sticky;
  top: 20px;
  align-self: start;
}
.preview-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 10px;
}
.preview-canvas {
  overflow: hidden;
  border: 1px solid #dcdfe6;
  border-radius: 8px;
  background: #f8f3ea;
  color: #2f3a3f;
  box-shadow: 0 10px 26px rgba(20, 28, 36, 0.08);
}
.preview-hero {
  position: relative;
  min-height: 260px;
  cursor: pointer;
  background: #e6e0d5;
}
.preview-hero img {
  width: 100%;
  height: 260px;
  object-fit: cover;
  display: block;
}
.image-placeholder {
  height: 260px;
  display: grid;
  place-items: center;
  color: #9c8f7c;
  font-size: 13px;
}
.hero-copy {
  position: absolute;
  left: 18px;
  right: 18px;
  bottom: 16px;
  color: white;
  text-shadow: 0 2px 12px rgba(0, 0, 0, 0.36);
}
.hero-copy p {
  margin: 0 0 8px;
  font-size: 11px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}
.hero-copy h3 {
  margin: 0;
  font-size: 28px;
  line-height: 1.05;
}
.hero-copy span {
  display: inline-block;
  margin-top: 10px;
  font-weight: 700;
}
.preview-block,
.preview-stop,
.preview-specs {
  padding: 16px 18px;
  border-top: 1px solid rgba(47, 58, 63, 0.12);
  cursor: pointer;
}
.preview-block h4,
.preview-stop h4 {
  margin: 0 0 8px;
  font-size: 16px;
}
.preview-block p,
.preview-stop p {
  margin: 0;
  color: #667176;
  line-height: 1.6;
  font-size: 13px;
  display: -webkit-box;
  -webkit-line-clamp: 4;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
.preview-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 6px;
  padding: 12px;
  border-top: 1px solid rgba(47, 58, 63, 0.12);
  cursor: pointer;
}
.thumb {
  aspect-ratio: 1;
  background: #e6e0d5;
  border-radius: 6px;
  overflow: hidden;
  display: grid;
  place-items: center;
  color: #9c8f7c;
  font-size: 12px;
}
.thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.preview-stop {
  display: grid;
  grid-template-columns: 54px 1fr;
  gap: 12px;
}
.preview-stop span {
  color: #b98033;
  font-weight: 700;
  font-size: 12px;
}
.preview-specs {
  display: grid;
  gap: 10px;
}
.preview-specs div {
  display: grid;
  grid-template-columns: 52px 1fr;
  gap: 10px;
}
.preview-specs span {
  color: #909399;
}
.preview-specs b {
  font-weight: 500;
}
@media (max-width: 1100px) {
  .template-preview {
    position: static;
  }
}
</style>
