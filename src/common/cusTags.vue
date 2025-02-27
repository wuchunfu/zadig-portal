<template>
  <div class="cus-tags" :class="{ 'cus-select': check }">
    <div v-if="noLimit">
      <span
        v-for="(value, index) in tags"
        :key="index"
        class="cut-tag"
        :class="{ current: index === currentTag,'no-border':noBorder?true:false,'no-limit':noLimit?true:false }"
        @click="toggleTag(index, value)"
      >{{ value }}</span>
    </div>
    <el-tooltip v-else effect="dark" placement="top-start">
      <div slot="content">{{values.toString()}}</div>
      <div>
        <span
          v-for="(value, index) in tags"
          :key="index"
          class="cut-tag"
          :class="{ current: index === currentTag,'no-border':noBorder?true:false}"
          @click="toggleTag(index, value)"
        >{{ value }}</span>
        <span class="cut-tag" v-if="values.length >2">...</span>
      </div>
    </el-tooltip>
  </div>
</template>

<script>
export default {
  name: 'CusTags',
  model: {
    prop: 'currentTag',
    event: 'updateCurrentTag'
  },
  props: {
    values: {
      type: Array,
      required: true
    },
    check: {
      default: false,
      type: Boolean
    },
    currentTag: {
      default: 0,
      type: Number
    },
    noBorder: {
      default: false,
      type: Boolean
    },
    noLimit: {
      default: false,
      type: Boolean
    }
  },
  computed: {
    tags () {
      if (this.noLimit) {
        return this.values
      } else {
        return this.values.slice(0, 2)
      }
    }
  },
  methods: {
    toggleTag (index, value) {
      if (this.check) {
        this.$emit('updateCurrentTag', index)
        this.$emit('toggleTag', { index, value })
      }
    }
  }
}
</script>

<style lang="less" scoped>
@tagColor: #888888;
@borderRadius: 4px;

.cus-tags {
  display: inline-block;
  color: @tagColor;
  font-size: 12px;
  white-space: nowrap;

  .cut-tag {
    position: relative;
    display: inline-block;
    max-width: 5em;
    padding: 0 1em;
    overflow: hidden;
    line-height: 2;
    text-overflow: ellipsis;
    vertical-align: middle;
    word-wrap: none;
    border: 1px solid @borderGray;

    &::before {
      position: absolute;
      top: 0.28em;
      left: -0.75em;
      width: 1.4em;
      height: 1.4em;
      border-top: 1px solid @borderGray;
      border-right: 1px solid @borderGray;
      transform: rotate(45deg);
      content: '';
    }

    &:first-child::before {
      border-width: 0;
    }

    &:not(:last-child) {
      padding-right: 0.7em;
      border-right-width: 0;
    }

    &:not(:first-child) {
      padding-left: 1.6em;
      border-left-width: 0;
    }

    &:first-child {
      border-top-left-radius: @borderRadius;
      border-bottom-left-radius: @borderRadius;
    }

    &:last-child {
      border-top-right-radius: @borderRadius;
      border-bottom-right-radius: @borderRadius;
    }

    &.no-border {
      padding: none;
      line-height: 1;
      border: none;

      &::before {
        top: 0;
        left: -0.75em;
        width: 1em;
        height: 1em;
      }

      &:not(:first-child) {
        padding-left: 1.2em;
      }
    }

    &.no-limit {
      max-width: unset;
    }
  }

  &.cus-select {
    cursor: pointer;

    &:hover {
      box-shadow: 0 0 2px @borderGray;
    }

    .cut-tag {
      &.current {
        color: @themeColor;
        border-color: @themeLightColor;

        &::before,
        + .cut-tag::before {
          border-color: @themeLightColor;
        }
      }
    }
  }
}
</style>
