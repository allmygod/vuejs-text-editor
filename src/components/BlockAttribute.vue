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
          <font-picker api-key="AIzaSyDQbiHQ2iamsHWa2XAAZgTj6VdQT0yK-6s" :active-font="attrs.fontFamily" @change="setFontFamily" :name="attrs.name" :id="'font-picker-'+attrs.name" />
        </div>
      </div>
      
      <!-- section -->
      <div class="block-attribute__detail__section">
        <div class="block-attribute__detail__section__title">
          FONT SIZE
        </div>
        <div class="block-attribute__detail__section__body">
          <select name="font-size" :value="attrs.fontSize" @change="setFontSize">
            <option value="8">8</option>
            <option value="9">9</option>
            <option value="10">10</option>
            <option value="12">12</option>
            <option value="14">14</option>
            <option value="16">16</option>
            <option value="18">18</option>
            <option value="20">20</option>
            <option value="24">24</option>
            <option value="28">28</option>
            <option value="32">32</option>
            <option value="36">36</option>
            <option value="48">48</option>
            <option value="72">72</option>
          </select>
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
    setFontSize(e) {
      this.$emit('attributeChanged', {
        blockName: this.attrs.name,
        attributeName: 'fontSize',
        attributeValue: parseInt(e.target.value)
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

