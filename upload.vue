<template>
  <div class="uploadFile">
    <el-button type="text" @click="triggerUpload" :disabled="uploading">上传</el-button>
    <input
      type="file"
      name="file"
      ref="uploadElement"
      @change="uploadFile"
      multiple="multiple"
      class="el-upload__input"
    />
    <span class="limit-error" v-if="limitError">{{msg}}</span>
  </div>
</template>

<script>
/* eslint-disable */
import { Message } from 'element-ui';
import download from './download';
import axios from 'axios';
export default {
  name: 'uploadFile',
  data() {
    return {
      uploading: false,
      limitError: false,
      msg: '',
    };
  },
  props: {},
  components: {
    download,
  },
  mounted() {},
  computed: {
    useId() {
      return -1;
    },
  },
  methods: {
    triggerUpload() {
      this.limitError = false;
      this.$refs.uploadElement.click();
    },
    uploadingTip(msg, tipMsg) {
      Message({
        message: msg,
        type: 'error',
        duration: 5000,
      });
      this.limitError = true;
      this.$refs.uploadElement.value = '';
      this.msg = tipMsg;
      return;
    },
    sendData(params) {
      console.log(params);
      this.uploading = true;
      const message = Message({
        message: '正在上传中，请稍等…………',
        type: 'info',
        duration: 0,
        center: true,
      });
      const url = '/customer/upload/uploadFiles';
      axios.defaults.headers.post['Content-Type'] = 'multipart/form-data';
      axios({
        url: url,
        method: 'post',
        data: params,
      })
        .then((res) => {
          this.uploading = false;
          message.close();
          console.log(res);
          if (res.data.data.success === true) {
            console.log(res);
          }
        })
        .catch((err) => {
          console.log(err);
          message.close();
          Message({
            message: '上传出错，请重试',
            type: 'error',
            duration: 2000,
          });
          this.$refs.uploadElement.value = '';
        });
    },
    sendFiles(option) {
      if (!option || !option.files || !option.files.length) {
        return;
      }
      const formData = new FormData();
      if (option.data) {
        Object.keys(option.data).forEach((key) => {
          formData.append(key, option.data[key]);
        });
      }
      for (let i = 0; i < option.files.length; i++) {
        formData.append('Filedata[]', option.files[i]);
      }
      this.sendData(formData);
    },
    uploadFile(e) {
      const canUploadType = /\.(pdf|xls|xlsx|csv|doc|docx|ppt|pptx|key|page|number)$/;
      const uploadELement = e.target || e.srcElement;
      const files = uploadELement.files;
      const canUploadArr = [];
      const unCanUploadArr = [];
      Array.from(files).forEach((file) => {
        if (!canUploadType.test(file.name)) {
          unCanUploadArr.push(file.name);
        } else {
          canUploadArr.push(file.name);
        }
      });
      if (unCanUploadArr.length > 0) {
        const msg = `${unCanUploadArr.join(',')}     文件类型不支持上传`;
        const tipMsg =
          '支持类型  .pdf  .xls  .xlsx  .csv  .doc  .docx  .ppt  .pptx  .key  .page  .number';
        this.uploadingTip(msg, tipMsg);
      }
      if (canUploadArr.length > 5) {
        const msg = '一次最多只能上传５个文件';
        const tipMsg = '一次最多只能上传５个文件';
        this.uploadingTip(msg, tipMsg);
      } else if (canUploadArr.length === 0) {
        return;
      } else if (canUploadArr.length < 5 && canUploadArr.length) {
        const isOverMaxFileSize = canUploadArr.some(
          (file) => file.size > AY.get('maxFileSize')
        );
        if (isOverMaxFileSize) {
          // 上传尺寸待定
          const msg = `上传的文件列表存在大于${AY.get('maxFileSize') /
            1024 /
            1024}M的文件，请重新上传`;
          const tipMsg = '文件过大';
          this.uploadingTip(msg, tipMsg);
        } else {
          this.sendFiles({
            files,
            data: {
              useId: this.useId,
            },
          });
        }
      }
    },
  },
};
</script>

<style>
.uploadFile .el-upload__input {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
}

.uploadFile .limit-error {
  margin-left: 5px;
  color: #ff4c4c;
  font-size: 14px;
}
</style>
