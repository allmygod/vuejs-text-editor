<template>
  <div class="block-attribute">
    <!-- header -->
    <div class="block-attribute__header">
      <a href="#" class="block-attribute__title" @click.prevent="toggleDetailView">
        <strong>{{abbrText}}</strong>
      </a>
      <a href="#" @click.prevent="handleDelete">
        <i class="fas fa-trash-alt" />
      </a>
    </div>

    <!-- detail -->
    <div class="block-attribute__detail" v-show="active && showDetail">
      <!-- section -->
      <div class="block-attribute__detail__section">
        <div class="block-attribute__detail__section__title">
          FONT FACE
        </div>
        <div class="block-attribute__detail__section__body">
          <font-picker api-key="replace-api-key-here" :active-font="attrs.fontFamily" @change="setFontFamily" />
        </div>
      </div>
      
      <!-- section -->
      <div class="block-attribute__detail__section">
        <div class="block-attribute__detail__section__title">
          FONT SIZE
        </div>
        <div class="block-attribute__detail__section__body">
        </div>
      </div>

      <!-- section -->
      <div class="block-attribute__detail__section">
        <div class="block-attribute__detail__section__title">
          ALIGN
        </div>
        <div class="block-attribute__detail__section__body">
          <a href="#" :class="attrs.align==='left' ? 'active' : ''" @click.prevent="setAlign('left')">
            <i class="fas fa-align-left" />
          </a>
          <a href="#" :class="attrs.align==='center' ? 'active' : ''" @click.prevent="setAlign('center')">
            <i class="fas fa-align-center" />
          </a>
          <a href="#" :class="attrs.align==='right' ? 'active' : ''" @click.prevent="setAlign('right')">
            <i class="fas fa-align-right" />
          </a>

          <a href="#" :class="attrs.verticalAlign==='top' ? 'active' : ''" @click.prevent="setVAlign('top')">
            <i class="fas fa-sort-up" />
          </a>
          <a href="#" :class="attrs.verticalAlign==='middle' ? 'active' : ''" @click.prevent="setVAlign('middle')">
            <i class="fas fa-sort" />
          </a>
          <a href="#" :class="attrs.verticalAlign==='bottom' ? 'active' : ''" @click.prevent="setVAlign('bottom')">
            <i class="fas fa-sort-down" />
          </a>
        </div>
      </div>

      <!-- section -->
      <div class="block-attribute__detail__section">
        <div class="block-attribute__detail__section__title">
          SIZE
        </div>
        <div class="block-attribute__detail__section__body">
          <input type="number" name="width" :value="attrs.cwidth" @change="setWidth" /> Wide by
          <input type="number" name="height" :value="attrs.cheight" @change="setHeight" /> tall
        </div>
      </div>
    </div>
    <!-- detail -->
  </div>
</template>

<script>
import FontPicker from 'font-picker-vue'

export default {
  props: {
    attrs: {
      type: Object
    },
    active: {
      type: Boolean
    }
  },
  components: {
    FontPicker
  },
  data() {
    return {
      showDetail: false,
    }
  },
  methods: {
    toggleDetailView() {
      this.showDetail = !this.showDetail
      this.$emit('activeBlockChanged', this.attrs.name)
    },
    setFontFamily(v) {
      this.$emit('attributeChanged', {
        blockName: this.attrs.name,
        attributeName: 'fontFamily',
        attributeValue: v.family
      })
    },
    setAlign(v) {
      this.$emit('attributeChanged', {
        blockName: this.attrs.name,
        attributeName: 'align',
        attributeValue: v
      })
    },
    setVAlign(v) {
      this.$emit('attributeChanged', {
        blockName: this.attrs.name,
        attributeName: 'verticalAlign',
        attributeValue: v
      })
    },
    setWidth(e) {
      this.$emit('attributeChanged', {
        blockName: this.attrs.name,
        attributeName: 'width',
        attributeValue: parseInt(e.target.value)
      })
    },
    setHeight(e) {
      this.$emit('attributeChanged', {
        blockName: this.attrs.name,
        attributeName: 'height',
        attributeValue: parseInt(e.target.value)
      })
    },
    handleDelete() {
      this.$emit('delete', this.attrs.name)
    }
  },
  computed: {
    abbrText() {
      return this.attrs.text.length < 20 ? this.attrs.text : this.attrs.text.substring(0, 19) + '...'
    }
  },
  watch: {
    active(val) {
      if (!val) {
        this.showDetail = false
      } else {
        this.showDetail = true
      }
    },
  }
}
</script>

