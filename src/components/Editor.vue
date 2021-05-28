<template>
  <div class="editor">
    <!-- editing region -->
    <div class="editor__wrapper">
      <v-stage
        name="stage"
        ref="stage"
        :config="{width, height}"
        @mousedown="handleMouseDown"
        @touchstart="handleMouseDown"
      >
        <v-layer ref="layer">
          <v-text
            v-for="block in blocks"
            :key="block.name"
            @dragend="handleDragEnd"
            @transform="handleTransform"
            @dblclick="handleInlineEdit"
            @dbltap="handleInlineEdit"
            :config="block"
          />
          <v-transformer
            ref="transformer"
            :rotateEnabled="false"
            :enabledAnchors="['middle-left', 'middle-right']"
            :borderStroke="selectedBorderColor"
            :anchorStroke="selectedBorderColor"
          />
        </v-layer>
      </v-stage>
    </div>

    <div class="editor__attributes">
      <!-- actions -->
      <div class="editor__attributes__actions">
        <button @click.prevent="addNewBlock">
          Add Text Block
        </button>

        <button @click.prevent="saveBlocks">
          Save
        </button>
      </div>

      <!-- content -->
      <div class="editor_attributes__content">
        <BlockAttribute
          v-for="block in blocks"
          :key="block.name"
          :attrs="block"
          :active="block.name===activeBlockName"
          @activeBlockChanged="handleActiveBlockChange"
          @attributeChanged="handleAttributeChange"
          @delete="handleDelete"
        />
      </div>
    </div>
  </div>
</template>

<script>
import BlockAttribute from './BlockAttribute.vue'
import '../assets/styles/index.scss'

const defaultFontFamily = 'Abel'
const defaultFontSize = 16
const selectedBorderColor='#00D2FF'
const defaultWidth = 100
const defaultHeight = defaultFontSize + 3

export default {
  components: {
    BlockAttribute
  },
  props: {
    width: {
      type: Number,
      default: 400
    },
    height: {
      type: Number,
      default: 300
    }
  },
  data() {
    return {
      selectedBorderColor,

      defaultBlockAttribute: {
        id: '',
        name: '',
        text: 'Type Here',
        
        x: 0,
        y: 0,
        width: defaultWidth,
        height: defaultHeight,

        draggable: true,
        fill: 'black',

        fontFamily: defaultFontFamily,
        fontSize: defaultFontSize,
        align: 'center',
        verticalAlign: 'top',

        // attributes, only for saving purpose
        cx: 0,
        cy: 0,
        cwidth: defaultWidth,
        cheight: defaultHeight,
      },
      blocks: [],
      activeBlockName: ''
    }
  },
  methods: {
    selectActiveBlock(name) {
      let self = this
      setTimeout(() => {
        self.activeBlockName = name
        self.updateTransformer()
      })
    },

    /**
     *  events of add / save blocks
     */
    addNewBlock() {
      let blockName = 'block' + Date.now()
      let newBlock = {...this.defaultBlockAttribute, name: blockName, id: blockName}
      if (this.blocks.length) {
        let lastBlock = this.blocks[this.blocks.length-1]
        newBlock = {...newBlock, fontFamily: lastBlock.fontFamily, fontSize: lastBlock.fontSize}
      }
      this.blocks.push(newBlock)

      this.selectActiveBlock(blockName)
    },
    saveBlocks() {
      let exportData = this.blocks.map(
        block => ({
          text: block.text,
          
          x: block.cx,
          y: block.cy,
          width: block.cwidth,
          height: block.cheight,

          fontFamily: block.fontFamily,
          fontSize: block.fontSize,

          align: block.align,
          verticalAlign: block.verticalAlign
        })
      )

      console.log(JSON.stringify(exportData))
    },

    /**
     *  events from attribute panels
     */
    handleActiveBlockChange(blockName) {
      this.selectActiveBlock(blockName)
    },
    handleAttributeChange(payload) {
      let block = this.blocks.find(b => b.name === payload.blockName)
      block[payload.attributeName] = payload.attributeValue

      // backup width, height info
      if (payload.attributeName === 'width' || payload.attributeName === 'height') {
        block['c' + payload.attributeName] = payload.attributeValue
      }
    },
    handleDelete(blockName) {
      this.selectActiveBlock('')

      let block = this.blocks.find(b => b.name === blockName)
      let index = this.blocks.indexOf(block)
      this.blocks.splice(index, 1)

      console.log('Deleted', block.text)
    },


    /**
     *  events from konva objects
     */
    handleDragEnd(e) {
      this.activeBlock.cx = parseInt(e.target.x())
      this.activeBlock.cy = parseInt(e.target.y())

      console.log(e.target.text(), `moved to ${this.activeBlock.cx},${this.activeBlock.cy}`)
    },
    handleMouseDown(e) {
      // if clicked on stage
      if (e.target.name() === 'stage') {
        this.activeBlockName = ''
        this.updateTransformer()
        return
      }

      // if clicked on transformer
      let clickedOnTransformer = e.target.getParent().className === 'Transformer'
      if (clickedOnTransformer) {
        return
      }

      // find clicked text-block
      let name = e.target.name()
      let block = this.blocks.find(b => b.name === name)
      if (block) {
        this.activeBlockName = name
      } else {
        this.activeBlockName = ''
      }
      this.updateTransformer()
    },
    updateTransformer() {
      // manually attach or detach transformer node
      let transformerNode = this.$refs.transformer.getNode()
      let stage = transformerNode.getStage()
      let selectedNode = stage.findOne('.' + this.activeBlockName)

      // if selected transformer node itself
      if (selectedNode === transformerNode.node()) {
        return
      }

      if (selectedNode) {
        transformerNode.nodes([selectedNode])
      } else {
        transformerNode.nodes([])
      }
    },
    handleTransform() {
      let stage = this.$refs.stage.getStage()
      let selectedNode = stage.findOne('.' + this.activeBlockName)

      selectedNode.setAttrs({
        width: selectedNode.width() * selectedNode.scaleX(),
        scaleX: 1
      })

      // save dimension
      this.activeBlock.cwidth = parseInt(selectedNode.width())
      this.activeBlock.cheight = parseInt(selectedNode.height())
    },
    handleInlineEdit() {
      let transformerNode = this.$refs.transformer.getNode()
      let stage = transformerNode.getStage()
      let textNode = stage.findOne('.' + this.activeBlockName)
      let textPosition = textNode.absolutePosition()
      let activeBlock = this.activeBlock

      transformerNode.hide()
      textNode.hide()
      
      let areaPosition = {
        x: stage.container().offsetLeft + textPosition.x,
        y: stage.container().offsetTop + textPosition.y,
      }

      // create textarea and style it
      let textarea = document.createElement('textarea');
      document.body.appendChild(textarea);

      // apply many styles to match text on canvas as close as possible
      // remember that text rendering on canvas and on the textarea can be different
      // and sometimes it is hard to make it 100% the same. But we will try...
      textarea.value = textNode.text();
      textarea.style.position = 'absolute';
      textarea.style.top = areaPosition.y + 'px';
      textarea.style.left = areaPosition.x + 'px';
      textarea.style.width = textNode.width() - textNode.padding() * 2 + 'px';
      textarea.style.height =
        textNode.height() - textNode.padding() * 2 + 5 + 'px';
      textarea.style.fontSize = textNode.fontSize() + 'px';
      textarea.style.border = '2px solid ' + selectedBorderColor;
      textarea.style.padding = '0px';
      textarea.style.margin = '0px';
      textarea.style.overflow = 'hidden';
      textarea.style.background = 'none';
      textarea.style.outline = 'none';
      textarea.style.resize = 'none';
      textarea.style.lineHeight = textNode.lineHeight();
      textarea.style.fontFamily = textNode.fontFamily();
      textarea.style.transformOrigin = 'left top';
      textarea.style.textAlign = textNode.align();
      textarea.style.color = textNode.fill();

      // rotation = textNode.rotation();
      let transform = '';
      // if (rotation) {
      //   transform += 'rotateZ(' + rotation + 'deg)';
      // }

      let px = 0;
      // also we need to slightly move textarea on firefox
      // because it jumps a bit
      let isFirefox =
        navigator.userAgent.toLowerCase().indexOf('firefox') > -1;
      if (isFirefox) {
        px += 2 + Math.round(textNode.fontSize() / 20);
      }
      transform += 'translateY(-' + px + 'px)';

      textarea.style.transform = transform;

      // reset height
      textarea.style.height = 'auto';
      // after browsers resized it we can set actual value
      textarea.style.height = textarea.scrollHeight + 3 + 'px';

      textarea.focus();


      function removeTextarea() {
        textarea.parentNode.removeChild(textarea);
        window.removeEventListener('click', handleOutsideClick);

        textNode.show();
        transformerNode.show();
        transformerNode.forceUpdate();
      }

      function setTextareaWidth(newWidth) {
        if (!newWidth) {
          // set width for placeholder
          newWidth = textNode.placeholder.length * textNode.fontSize();
        }
        // some extra fixes on different browsers
        let isSafari = /^((?!chrome|android).)*safari/i.test(
          navigator.userAgent
        );
        let isFirefox =
          navigator.userAgent.toLowerCase().indexOf('firefox') > -1;
        if (isSafari || isFirefox) {
          newWidth = Math.ceil(newWidth);
        }

        let isEdge =
          document.documentMode || /Edge/.test(navigator.userAgent);
        if (isEdge) {
          newWidth += 1;
        }
        textarea.style.width = newWidth + 'px';
      }

      textarea.addEventListener('keydown', function (e) {
        // hide on enter
        // but don't hide on shift + enter
        if (e.keyCode === 13 && !e.shiftKey) {
          textNode.text(textarea.value);
          activeBlock.text = textarea.value
          removeTextarea();
        }
        // on esc do not set value back to node
        if (e.keyCode === 27) {
          removeTextarea();
        }
      });

      textarea.addEventListener('keydown', function () {
        // let scale = textNode.getAbsoluteScale().x;
        setTextareaWidth(textNode.width());
        textarea.style.height = 'auto';
        textarea.style.height =
          textarea.scrollHeight + textNode.fontSize() + 'px';
      });

      function handleOutsideClick(e) {
        if (e.target !== textarea) {
          textNode.text(textarea.value);
          activeBlock.text = textarea.value
          removeTextarea();
        }
      }

      setTimeout(() => {
        window.addEventListener('click', handleOutsideClick);
      });
    }
  },
  computed: {
    activeBlock() {
      return this.blocks.find(b => b.name === this.activeBlockName)
    }
  }
}
</script>
