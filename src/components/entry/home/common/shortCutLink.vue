<template>
  <div class="shortcut-link-container">
    <el-popover placement="bottom" popper-class="shortcuts-droplist" trigger="hover">
      <ul class="dropdown-menu">
        <span class="title">快捷链接</span>
        <li role="separator" class="divider"></li>
        <template v-if="links.length > 0">
          <li v-for="(link, index) in links" :key="index" class="link-container">
            <a :href="link.url" target="_blank">
              <span class="icon">
                <img src="@assets/icons/others/link.svg" />
              </span>
              <div class="info-wrap">
                <span class="link-name">{{link.name}}</span>
                <span class="link-url">{{link.url}}</span>
              </div>
            </a>
          </li>
        </template>
        <li v-else class="no-link">
          <span class="icon">
            <img src="@assets/icons/others/link.svg" />
          </span>
          <span class="desc">暂无快捷链接</span>
        </li>
      </ul>
      <span slot="reference">
        <i class="function-icon iconfont iconkaifang"></i>
      </span>
    </el-popover>
  </div>
</template>
<script>
import { getExternalLinksAPI } from '@api'
import { mapState } from 'vuex'
export default {
  computed: {
    ...mapState({
      links: state => state.externalLink.links
    })
  },
  methods: {
    getExternalLinks () {
      getExternalLinksAPI().then(res => {
        this.$store.commit('SET_EXTERNAL_LINK', res || [])
      })
    }
  },
  created () {
    this.getExternalLinks()
  }
}
</script>
<style lang="less">
.shortcuts-droplist {
  min-width: 0;

  .dropdown-menu {
    display: flex;
    flex-direction: column;
    padding: 4px 0;
    padding-bottom: 0;
    list-style: none;

    .divider {
      height: 1px;
      margin: 9px 0;
      padding: 0;
      overflow: hidden;
      background-color: #e5e5e5;
    }

    .title {
      display: flex;
    }

    li {
      display: flex;
    }

    .no-link {
      display: flex;
      align-items: center;
      padding: 60px 40px;

      .icon {
        margin-right: 5px;

        img {
          width: 18px;
          height: 18px;
        }
      }

      .desc {
        color: #c0c4cc;
        font-size: 14px;
      }
    }

    .link-container {
      display: flex;
      flex-direction: column;
      padding: 8px 10px;
      border-radius: 6px;

      &:hover {
        background-color: rgba(0, 102, 255, 0.07);
      }

      a {
        display: flex;
        align-items: center;

        .icon {
          display: flex;
          margin-right: 10px;

          img {
            width: 24px;
            height: 24px;
          }
        }

        .info-wrap {
          display: flex;
          flex-direction: column;

          .link-name {
            display: flex;
            color: rgb(16, 16, 16);
            font-weight: 500;
            font-size: 13px;
          }

          .link-url {
            display: flex;
            color: #909399;
            font-size: 12px;
          }
        }
      }
    }
  }
}

.shortcut-link-container {
  .function-icon {
    color: #a0a0a0;
    font-size: 20px;
    cursor: pointer;

    &:hover {
      color: @themeColor;
    }
  }
}
</style>
