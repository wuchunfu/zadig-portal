<template>
  <section class="other-step-container">
    <el-drawer :modal-append-to-body="false" title="Dockerfile 模板预览" :visible.sync="showDockerfile" direction="rtl">
      <Codemirror
        v-model="dockerfileTemplate.content"
        class="mirror"
        :cmOption="{
      tabSize: 2,
      readOnly: true,
      theme: 'neo',
      mode: 'text/x-dockerfile',
      lineNumbers: false,
      line: true,
      showGutter: false,
      displayIndentGuides: false,
      showPrintMargin: false,
      collapseIdentical: true
    }"
     />
    </el-drawer>
    <div style="margin: 14px 0 8px;">
      <el-dropdown @command="addExtra">
        <el-button type="primary" size="small" plain>
          {{$t(`build.addSteps`)}}
          <i style="margin-left: 8px;" class="el-icon-arrow-down el-icon--right"></i>
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item command="docker" :disabled="docker_enabled">{{$t(`build.stepImageBuild`)}}</el-dropdown-item>
          <el-dropdown-item command="binary" :disabled="binary_enabled">{{$t(`build.stepArtifactUpload`)}}</el-dropdown-item>
          <el-dropdown-item command="object" :disabled="object_storage_upload_enabled">{{$t(`build.stepFileUpload`)}}</el-dropdown-item>
          <el-dropdown-item command="script" v-if="!usedToHost" :disabled="post_script_enabled">{{$t(`build.stepRunShellScript`)}}</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
    <div class="common-parcel-block" v-if="docker_enabled || binary_enabled || object_storage_upload_enabled ||post_script_enabled">
      <el-form
        v-if="docker_enabled && buildConfig.post_build.docker_build"
        :model="buildConfig.post_build.docker_build"
        :rules="docker_rules"
        ref="dockerBuildRef"
        label-width="170px"
        class="secondary-form"
        :label-position="mini ? 'top' : 'left'"
      >
        <div class="dashed-container">
          <span class="primary-title">
            {{$t(`build.stepImageBuild`)}}
            <el-button type="text" @click="removeDocker" icon="el-icon-delete"></el-button>
          </span>
          <div v-if="!usedToHost && allRegistry.length === 0" class="registry-alert">
            <el-alert :title="$t(`build.prompt.noImageRegistry`)" type="warning"></el-alert>
          </div>
          <el-form-item :label="$t(`build.buildContextDirectory`)" prop="work_dir">
            <el-input v-model="buildConfig.post_build.docker_build.work_dir" size="small">
              <template slot="prepend">$WORKSPACE/</template>
            </el-input>
          </el-form-item>
          <el-form-item :label="$t(`build.dockerfileSource`)" prop="source">
            <el-select size="small" v-model="buildConfig.post_build.docker_build.source" :placeholder="$t(`build.prompt.select`)" @change="$refs.dockerBuildRef.clearValidate()">
              <el-option :label="$t(`global.repository`)" value="local"></el-option>
              <el-option :label="$t(`global.templates`)" value="template"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item v-if="buildConfig.post_build.docker_build.source === 'local'" :label="$t(`build.dockerfileAbsolutePath`)" prop="docker_file">
            <el-input v-model="buildConfig.post_build.docker_build.docker_file" size="small">
              <template slot="prepend">$WORKSPACE/</template>
            </el-input>
          </el-form-item>
          <el-form-item v-if="buildConfig.post_build.docker_build.source === 'template'" :label="$t(`build.prompt.select`)" prop="template_id">
            <el-select
              style="width: 90%;"
              size="small"
              filterable
              @change="getDockerfileTemplate"
              v-model="buildConfig.post_build.docker_build.template_id"
              :placeholder="$t(`build.prompt.select`)"
            >
              <el-option v-for="(template,index) in dockerfileTemplates" :key="index" :label="template.name" :value="template.id"></el-option>
            </el-select>
            <template>
              <el-button
                :disabled="!buildConfig.post_build.docker_build.template_id"
                style="margin-left: 5px;"
                type="text"
                @click="showDockerfile = true"
              >{{$t(`build.preview`)}}</el-button>
              <div v-if="dockerfileTemplate.variable && dockerfileTemplate.variable.length > 0" class="dockerfile-args-container">
                <span>ARG</span>
                <span v-for="(item,index) in dockerfileTemplate.variable" :key="index">
                  <span v-if="item.value">{{`${item.key}=${item.value} `}}</span>
                  <span v-else>{{`${item.key} `}}</span>
                </span>
              </div>
            </template>
          </el-form-item>
          <el-form-item :label="$t(`build.dockerBuildArgs`)">
            <el-tooltip effect="dark" :content="$t(`build.prompt.dockerBuildArgs`)" placement="top-start">
              <el-input v-model="buildConfig.post_build.docker_build.build_args" size="small" placeholder="--build-arg key=value"></el-input>
            </el-tooltip>
          </el-form-item>
        </div>
      </el-form>
      <el-form
        v-if="binary_enabled && buildConfig.post_build.file_archive"
        :model="buildConfig.post_build.file_archive"
        :rules="file_archive_rules"
        ref="fileArchiveRef"
        label-width="170px"
        class="secondary-form"
        :label-position="mini ? 'top' : 'left'"
      >
        <div class="dashed-container">
          <span class="primary-title">
            {{$t(`build.stepArtifactUpload`)}}
            <el-button type="text" @click="removeBinary" icon="el-icon-delete"></el-button>
          </span>
          <el-form-item :label="$t(`build.artifactPath`)" prop="file_location">
            <el-input v-model="buildConfig.post_build.file_archive.file_location" size="small">
              <template slot="append">/$PKG_FILE</template>
              <template slot="prepend">$WORKSPACE/</template>
            </el-input>
          </el-form-item>
        </div>
      </el-form>
    <el-form
        v-if="object_storage_upload_enabled && buildConfig.post_build.object_storage_upload"
        :model="buildConfig.post_build.object_storage_upload"
        :rules="object_storage_rules"
        ref="objectStorageRef"
        label-width="170px"
        class="secondary-form"
        :label-position="mini ? 'top' : 'left'"
      >
        <div class="dashed-container">
          <span class="primary-title">
            {{$t(`build.stepFileUpload`)}}
            <el-button type="text" @click="removeObject" icon="el-icon-delete"></el-button>
          </span>
          <el-form-item :label="$t(`sysSetting.integration.storageTab`)" prop="object_storage_id">
            <el-select size="small" v-model="buildConfig.post_build.object_storage_upload.object_storage_id" :placeholder="$t(`build.prompt.selectObjectStorage`)" @change="$refs.objectStorageRef.clearValidate()">
              <el-option v-for="(item,index) in objectStorageList" :key="index" :label="`${item.endpoint}/${item.bucket}`" :value="item.id"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item :label="$t(`build.uploadFile`)" prop="upload_detail">
            <template v-if="buildConfig.post_build.object_storage_upload.upload_detail.length > 0" >
              <el-row v-for="(item,index) in buildConfig.post_build.object_storage_upload.upload_detail" :key="index">
                <el-col :span="11">
                    <el-input v-model="item.file_path" style="max-width: 100%;" size="small">
                      <template slot="prepend">$WORKSPACE/</template>
                    </el-input>
                </el-col>
                <el-col :span="1" style="text-align: center;">to</el-col>
                <el-col :span="8">
                    <el-input v-model="item.dest_path" size="small">
                    </el-input>
                </el-col>
                <el-col :span="4">
                  <div class="">
                    <el-button @click="removeObjectStorage(index)" type="danger" icon="el-icon-minus"  size="mini" circle plain></el-button>
                    <el-button v-if="index === buildConfig.post_build.object_storage_upload.upload_detail.length - 1" type="primary" icon="el-icon-plus"  @click="addObjectStorage" size="mini" circle plain></el-button>
                  </div>
                </el-col>
              </el-row>
            </template>
            <el-button v-else type="plain" icon="el-icon-plus" @click="addObjectStorage" size="mini" circle plain></el-button>
          </el-form-item>
        </div>
      </el-form>
      <el-form
        v-if="post_script_enabled && buildConfig.post_build.scripts"
        :model="buildConfig.post_build"
        ref="scriptRef"
        label-width="220px"
        class="stcov label-at-left"
      >
        <div class="dashed-container">
          <div class="primary-title">
            {{$t(`build.stepRunShellScript`)}}
            <el-tooltip effect="dark" :content="$t(`build.prompt.runShellScriptDescription`)" placement="top">
              <i class="el-icon-question" style="color: #a0a0a0;"></i>
            </el-tooltip>
            <el-button type="text" @click="removeScript" icon="el-icon-delete"></el-button>
          </div>
          <div class="script-content">
            <Editor v-model="buildConfig.post_build.scripts" height="100%" />
          </div>
        </div>
      </el-form>
    </div>
  </section>
</template>

<script>
import Editor from 'vue2-ace-bind'
import Codemirror from '@/components/projects/common/codemirror.vue'
import {
  getDockerfileAPI,
  getDockerfileTemplatesAPI,
  getRegistryWhenBuildAPI,
  getStorageListAPI
} from '@api'
export default {
  props: {
    buildConfig: Object,
    validObj: Object,
    usedToHost: Boolean, // Cloud hosting builds do not require mirror repositories
    mini: Boolean
  },
  data () {
    return {
      allRegistry: [],
      dockerfileTemplates: [],
      objectStorageList: [],
      showDockerfile: false,
      object_storage_upload_enabled: false,
      post_script_enabled: false,
      docker_enabled: false,
      binary_enabled: false,
      dockerfileTemplate: {}
    }
  },
  methods: {
    // called by the parent component at edit time
    initStepStatus (buildConfig = this.buildConfig) {
      if (buildConfig.post_build.docker_build) {
        this.docker_enabled = true
        if (buildConfig.post_build.docker_build.template_id) {
          this.getDockerfileTemplate(
            buildConfig.post_build.docker_build.template_id
          )
        }
      } else {
        this.docker_enabled = false
      }
      if (buildConfig.post_build.file_archive) {
        this.binary_enabled = true
      } else {
        this.binary_enabled = false
      }
      if (buildConfig.post_build.object_storage_upload) {
        this.object_storage_upload_enabled = true
      } else {
        this.object_storage_upload_enabled = false
      }
      if (buildConfig.post_build.scripts) {
        this.post_script_enabled = true
      } else {
        this.post_script_enabled = false
      }
    },
    addExtra (command) {
      if (command === 'docker') {
        this.docker_enabled = true
        this.$set(this.buildConfig.post_build, 'docker_build', {
          work_dir: '',
          docker_file: '',
          build_args: '',
          source: 'local'
        })
      }
      if (command === 'binary') {
        this.binary_enabled = true
        this.$set(this.buildConfig.post_build, 'file_archive', {
          file_location: ''
        })
      }
      if (command === 'script') {
        this.post_script_enabled = true
        this.$set(this.buildConfig.post_build, 'scripts', '#!/bin/bash\nset -e')
      }
      if (command === 'object') {
        this.object_storage_upload_enabled = true
        this.$set(this.buildConfig.post_build, 'object_storage_upload', {
          enabled: true,
          object_storage_id: '',
          upload_detail: []
        })
      }
      this.$nextTick(() => {
        document.querySelector('.other-step-container').scrollIntoView({
          behavior: 'smooth'
        })
      })
    },
    removeDocker () {
      this.docker_enabled = false
      delete this.buildConfig.post_build.docker_build
    },
    removeBinary () {
      this.binary_enabled = false
      delete this.buildConfig.post_build.file_archive
    },
    removeScript () {
      this.post_script_enabled = false
      delete this.buildConfig.post_build.scripts
    },
    removeObject () {
      this.object_storage_upload_enabled = false
      delete this.buildConfig.post_build.object_storage_upload
    },
    addObjectStorage () {
      this.buildConfig.post_build.object_storage_upload.upload_detail.push({
        file_path: '',
        dest_path: ''
      })
    },
    removeObjectStorage (index) {
      this.buildConfig.post_build.object_storage_upload.upload_detail.splice(index, 1)
    },
    async getDockerfileTemplate (id) {
      const res = await getDockerfileAPI(id).catch(err => {
        console.log(err)
      })
      if (res) {
        this.dockerfileTemplate = res
      }
    },
    getUsedData () {
      const projectName = this.$route.params.project_name
      if (!this.usedToHost) {
        getRegistryWhenBuildAPI(projectName).then(res => {
          this.allRegistry = res
        })
      }
      getDockerfileTemplatesAPI(projectName).then(res => {
        this.dockerfileTemplates = res.dockerfile_template
      })
      const key = this.$utils.rsaEncrypt()
      getStorageListAPI(key).then(res => {
        this.objectStorageList = res
      })
    },
    validate () {
      const valid = []
      if (this.docker_enabled) {
        valid.push(this.$refs.dockerBuildRef.validate())
      }
      if (this.binary_enabled) {
        valid.push(this.$refs.fileArchiveRef.validate())
      }

      if (this.object_storage_upload_enabled) {
        valid.push(this.$refs.objectStorageRef.validate())
      }
      return Promise.all(valid)
    }
  },
  computed: {
    docker_rules () {
      return {
        work_dir: [
          {
            type: 'string',
            message: this.$t(`build.prompt.fillInDockerBuildWorkingDirectory`),
            required: true,
            trigger: 'blur'
          }
        ],
        docker_file: [
          {
            type: 'string',
            message: this.$t(`build.prompt.fillInDockerfilePath`),
            required: true,
            trigger: 'blur'
          }
        ],
        template_id: [
          {
            type: 'string',
            message: this.$t(`build.prompt.selectInDockerfileTemplate`),
            required: true,
            trigger: ['blur', 'change']
          }
        ]
      }
    },
    file_archive_rules () {
      return {
        file_location: [
          {
            type: 'string',
            message: this.$t(`build.prompt.fillInArchiveFilePath`),
            required: true,
            trigger: 'blur'
          }
        ]
      }
    },
    object_storage_rules () {
      return {
        object_storage_id: [
          {
            type: 'string',
            message: this.$t(`build.prompt.selectObjectStorage`),
            required: true,
            trigger: 'blur'
          }
        ],
        upload_detail: [
          {
            type: 'array',
            required: true,
            validator: (rule, value, callback) => {
              const empty = value.every(item => {
                return !item.file_path || !item.dest_path
              })
              if (value.length === 0) {
                callback(new Error(this.$t(`build.prompt.atLeastUploadOneFile`)))
              } else if (empty) {
                callback(new Error(this.$t(`build.prompt.uploadFilePathEmpty`)))
              } else {
                callback()
              }
            }
          }
        ]
      }
    }
  },
  created () {
    this.getUsedData()
    this.validObj.addValidate({
      name: 'otherStepsValid',
      valid: this.validate
    })
  },
  components: {
    Editor,
    Codemirror
  }
}
</script>

<style lang="less" scoped>
@inputWidth: 400px;
@secondaryColor: #888888;
@primaryColor: #000;
@formItemBottom: 8px;
@labelWeight: 300;

.other-step-container {
  .common-parcel-block {
    .primary-title {
      margin: 0;
      line-height: 28px;
    }

    /deep/.el-form.secondary-form {
      .el-input,
      .el-select {
        width: 100%;
      }

      .el-form-item {
        margin-bottom: 15px;
      }
    }
  }

  /deep/.el-form-item__content {
    .el-input-group {
      vertical-align: baseline;
    }
  }

  .script-content {
    height: 250px;
    border: 1px solid @borderGray;
  }

  .registry-alert {
    margin-bottom: 10px;
  }
}
</style>
