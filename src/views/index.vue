<template>
  <div class="app-container home">
    <el-row :gutter="20">
      <el-col :sm="24" :lg="12" style="padding-left: 20px">
        <h2>YYY后台管理框架</h2>

        <p>
          <el-button type="primary" size="mini" icon="el-icon-cloudy" plain>测试按钮1</el-button>
          <el-button size="mini" icon="el-icon-s-home" plain>测试按钮2</el-button>
        </p>
      </el-col>
    </el-row>
    <el-divider />


    <p>
      <el-button type="info" plain icon="el-icon-upload2" size="mini" @click="handleImport">导入</el-button>
      <!-- 用户导入对话框 -->
      <el-dialog :title="upload.title" :visible.sync="upload.open" @closed="handleClose" width="400px" append-to-body>
        <el-upload ref="upload" :limit="1" accept=".xlsx, .xls" :headers="upload.headers" :action="upload.url"
          :disabled="upload.isUploading" :on-progress="handleFileUploadProgress" :on-success="handleFileSuccess"
          :auto-upload="false" drag>
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
          <div class="el-upload__tip text-center" slot="tip">
            <span>仅允许导入xls、xlsx格式文件。</span>
            <br />
          </div>
        </el-upload>
        <div v-if="uploadResult">
          <p>成功数量: {{ uploadResult.successCount || 0 }}</p>
          <p>失败数量: {{ uploadResult.failedCount || 0 }}</p>
          <div v-if="uploadResult.failedCount > 0">
            <p>失败文件下载: <el-link type="primary" :underline="false" style="font-size:12px;vertical-align: baseline;"
                @click="downloadFaileFile">点击下载</el-link></p>
          </div>
        </div>
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="submitFileForm">确 定</el-button>
          <el-button @click="upload.open = false">取 消</el-button>
        </div>
      </el-dialog>

      <el-button type="primary" style="margin-left: 10px" plain icon="el-icon-cloudy" size="mini"
        @click="openDialog">Open Form Dialog</el-button>
      <!-- 测试多选、上传、单选表单提交对话框 -->
      <el-dialog title="Form Dialog" :visible.sync="dialogVisible">
        <el-form :model="form" ref="form" label-width="100px">
          <el-form-item label="Upload Files">
            <el-upload class="upload-demo" action="" :show-file-list="true" :on-change="handleFileChange"
              :before-upload="beforeUpload" :auto-upload="false" :multiple="true">
              <el-button size="small" type="primary">Click to Upload</el-button>
            </el-upload>
          </el-form-item>

          <el-form-item label="Multi Select">
            <el-select v-model="form.multiSelect" multiple placeholder="Please select">
              <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>

          <el-form-item label="Radio">
            <el-radio-group v-model="form.radio">
              <el-radio :label="1">Option One</el-radio>
              <el-radio :label="2">Option Two</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">Cancel</el-button>
          <el-button type="primary" @click="submitForm('form')">Submit</el-button>
        </span>
      </el-dialog>
    </p>

  </div>
</template>

<script>
import request from '@/utils/request'
import { getToken } from "@/utils/auth";
import { getBase64Type, base64toBlob } from "@/utils/index";
export default {
  name: "Index",
  data() {
    return {
      // 用户导入参数
      upload: {
        // 是否显示弹出层（用户导入）
        open: false,
        // 弹出层标题（用户导入）
        title: "",
        // 是否禁用上传
        isUploading: false,
        // 设置上传的请求头部
        headers: { Authorization: "Bearer " + getToken() },
        // 上传的地址
        url: process.env.VUE_APP_BASE_API + "/system/user/importUserByExcel"
      },
      // 用户导入统计结果
      uploadResult: null,
      failedFilesLink: '',


      dialogVisible: false,
      form: {
        multiSelect: [],
        radio: '',
        files: []
      },
      options: [{
        value: 'option1',
        label: 'Option 1'
      }, {
        value: 'option2',
        label: 'Option 2'
      }, {
        value: 'option3',
        label: 'Option 3'
      }]
    };
  },
  methods: {
    /** 导入按钮操作 */
    handleImport() {
      this.upload.title = "用户导入";
      this.upload.open = true;
    },
    /** 下载模板操作 */
    importTemplate() {
      this.download('system/user/importTemplate', {
      }, `user_template_${new Date().getTime()}.xlsx`)
    },
    // 文件上传中处理
    handleFileUploadProgress(event, file, fileList) {
      this.upload.isUploading = true;
    },
    // 文件上传成功处理
    handleFileSuccess(response, file, fileList) {
      if (response.code == '200') {
        this.upload.isUploading = false;
        this.$refs.upload.clearFiles();
        // console.log('~~~~response', response)
        // 假设response包含了成功和失败的数量及失败文件
        this.uploadResult = {
          successNum: response.data.successNum,
          failedCount: response.data.errorNum,
        };
        if (this.uploadResult.failedCount > 0) {
          this.failedFilesLink = response.data.content
        }
      } else {
        this.upload.open = false;
        this.upload.isUploading = false;
        this.$refs.upload.clearFiles();
        this.$modal.msgError(response.msg);
      }
    },
    // 提交上传文件
    submitFileForm() {
      this.$refs.upload.submit();
    },
    downloadFaileFile() {
      let base64url = getBase64Type('xlsx') + this.failedFilesLink
      var blob = base64toBlob(base64url)

      // 这里就是创建一个a标签，等下用来模拟点击事件
      let downloadElement = document.createElement("a");
      // 根据解析后的blob对象创建URL 对象
      let href = window.URL.createObjectURL(blob);
      // 下载链接
      downloadElement.href = href;
      // 下载文件名,如果后端没有返回，可以自己写a.download = '文件.pdf'
      downloadElement.download = "用户导入失败.xlsx";
      document.body.appendChild(downloadElement);
      // 点击a标签，进行下载
      downloadElement.click();
      // 收尾工作，在内存中移除URL 对象
      window.URL.revokeObjectURL(href);
    },
    handleClose() {
      this.uploadResult = null
      this.failedFilesLink = ''
    },


    openDialog() {
      this.dialogVisible = true;
    },
    handleFileChange(file, fileList) {
      this.form.files.push(file.raw);
    },
    beforeUpload(file) {
      console.log(file);
      return true; // 返回false可以阻止文件上传
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          //要发送文件和非文件数据，最简单的方法是使用 FormData 直接发送所有数据 后端不用写任何注解，创建一个vo接收即可
          const formData = new FormData();  
          this.form.files.forEach(file => {
            formData.append('files', file);
          });
          this.form.multiSelect.forEach(item => {
            formData.append('multiSelect', item);
          });
          formData.append('radio', this.form.radio);

          //否则只有非文件的数据时，我们也可以使用FormData方式 后端不用写任何注解，创建一个vo接收即可  或者 json方式(直接传this.form即可) 后端用@RequestBody接收

          // 使用axios或其他HTTP客户端发送表单数据到服务器
          request.post('/test/test/getFromRequest', formData)
            .then(response => {
              this.dialogVisible = false;
            })
            .catch(error => {
              console.error(error);
            });
        } else {
          console.log('error submit!!');
          return false;
        }
      });
    }
  },
};
</script>

<style scoped lang="scss">
.home {
  blockquote {
    padding: 10px 20px;
    margin: 0 0 20px;
    font-size: 17.5px;
    border-left: 5px solid #eee;
  }

  hr {
    margin-top: 20px;
    margin-bottom: 20px;
    border: 0;
    border-top: 1px solid #eee;
  }

  .col-item {
    margin-bottom: 20px;
  }

  ul {
    padding: 0;
    margin: 0;
  }

  font-family: "open sans",
  "Helvetica Neue",
  Helvetica,
  Arial,
  sans-serif;
  font-size: 13px;
  color: #676a6c;
  overflow-x: hidden;

  ul {
    list-style-type: none;
  }

  h4 {
    margin-top: 0px;
  }

  h2 {
    margin-top: 10px;
    font-size: 26px;
    font-weight: 100;
  }

  p {
    margin-top: 10px;

    b {
      font-weight: 700;
    }
  }

  .update-log {
    ol {
      display: block;
      list-style-type: decimal;
      margin-block-start: 1em;
      margin-block-end: 1em;
      margin-inline-start: 0;
      margin-inline-end: 0;
      padding-inline-start: 40px;
    }
  }
}
</style>
