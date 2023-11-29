<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button class="filter-item" style="margin-left: 10px" type="primary" icon="el-icon-edit" @click="handleCreate">
        {{ $t("table.add") }}
      </el-button>
    </div>

    <el-table :key="tableKey" v-loading="listLoading" :data="list" stripe fit highlight-current-row style="width: 100%"
      @sort-change="sortChange" @selection-change="handleSelection"
      :default-sort="{ prop: 'create_time', order: 'descending' }">
      <el-table-column label="所属类别" prop="type">
        <template slot-scope="scope">
          <span style="color: #53d6c4; cursor: pointer" v-if="scope.row.type == 1">轮播</span>
          <span style="color: #53d6c4; cursor: pointer" v-if="scope.row.type == 2">公告</span>
          <span style="color: #53d6c4; cursor: pointer" v-if="scope.row.type == 3">系统通知</span>
        </template>
      </el-table-column>
      <el-table-column label="标题" min-width="120" prop="title">
        <template slot-scope="scope">
          <el-tooltip :content="scope.row.title">
            <span v-if="scope.row.title ? scope.row.title.length > 8 : false">
              {{ scope.row.title.slice(0, 8) }}...
            </span>
            <span v-else> {{ scope.row.title }} </span>
          </el-tooltip>
        </template>
      </el-table-column>


      <el-table-column label="缩略图">
        <template slot-scope="scope">
          <el-image style="width: 50px; height: 50px" @click="see_img(scope.row.img)" :src="scope.row.img" fit="cover"
            lazy>
            <div slot="error" class="image-slot">
              <i class="el-icon-picture-outline" style="font-size: 330%"></i>
            </div>
          </el-image>
        </template>
      </el-table-column>
      <el-table-column label="状态" prop="status">
        <template slot-scope="scope">
          <span style="color: #53d6c4; cursor: pointer" v-if="scope.row.status == 1">启用</span>
          <span style="color: #ff643a; cursor: pointer" v-if="scope.row.status == 0">禁用</span>
        </template>
      </el-table-column>

      <el-table-column label="创建时间" min-width="130" prop="create_time">
      </el-table-column>
      <el-table-column label="操作" prop="statue">
        <template slot-scope="scope">
          <span style="color: #ff643a; cursor: pointer" @click="del_row(scope.row.id)">删除</span>
          |
          <span style="color: #1890ff; cursor: pointer" @click="edit_row(scope.row)">编辑</span>
        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total > 0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.pagesize"
      @pagination="getList" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible" width="90%">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="120px">
        <div style="width: 100%; margin-left: 50px">
          <div style="width: 90%;">
            <el-form-item label="标题" prop="title">
              <el-tabs v-model="activeName" type="border-card">

                <el-tab-pane label="繁体中文" name="zh" > <el-input v-model="temp.title" /></el-tab-pane>
                <el-tab-pane label="简体中文" name="zhCN" v-if="get_yuyan.includes('zhCN')"> <el-input v-model="temp.title1" /></el-tab-pane>
                <el-tab-pane label="英文" name="en" v-if="get_yuyan.includes('en')"> <el-input v-model="temp.title2" /></el-tab-pane>
                <el-tab-pane label="日文" name="japan" v-if="get_yuyan.includes('japan')"> <el-input v-model="temp.title3" /></el-tab-pane>
                <el-tab-pane label="韩文" name="han" v-if="get_yuyan.includes('han')"> <el-input v-model="temp.title4" /></el-tab-pane>
                <el-tab-pane label="法语" name="french" v-if="get_yuyan.includes('french')"> <el-input v-model="temp.title5" /></el-tab-pane>
                <el-tab-pane label="德语" name="de" v-if="get_yuyan.includes('de')"> <el-input v-model="temp.title6" /></el-tab-pane>
                <el-tab-pane label="西班牙语" name="xby" v-if="get_yuyan.includes('xby')"> <el-input v-model="temp.title7" /></el-tab-pane>
                <el-tab-pane label="俄语" name="eyu" v-if="get_yuyan.includes('eyu')"> <el-input v-model="temp.title8" /></el-tab-pane>

              </el-tabs>
            </el-form-item>
          </div>
          <div style="width: 500px;">
            <el-form-item label="所属类别">
              <el-select v-model="temp.type_id" class="filter-item" style="width: 100%" placeholder="Please select">
                <el-option label="轮播图" :value="1" />
                <el-option label="公告" :value="2" />
                <el-option label="系统通知" :value="3" />
              </el-select>
            </el-form-item>
            <el-form-item label="状态">
              <el-radio v-model="temp.status" :label="1">启用</el-radio>
              <el-radio v-model="temp.status" :label="0">禁用</el-radio>
            </el-form-item>
          </div>
          <div style="width: 90%;">
            <el-form-item label="描述" prop="title">
              <el-tabs v-model="activeName" type="border-card">

                <el-tab-pane label="繁体中文" name="zh"> <el-input v-model="temp.desc" /></el-tab-pane>
                <el-tab-pane label="简体中文" name="zhCN" v-if="get_yuyan.includes('zhCN')"> <el-input v-model="temp.desc1" /></el-tab-pane>
                <el-tab-pane label="英文" name="en" v-if="get_yuyan.includes('en')"> <el-input v-model="temp.desc2" /></el-tab-pane>
                <el-tab-pane label="日文" name="japan" v-if="get_yuyan.includes('japan')"> <el-input v-model="temp.desc3" /></el-tab-pane>
                <el-tab-pane label="韩文" name="han" v-if="get_yuyan.includes('han')"> <el-input v-model="temp.desc4" /></el-tab-pane>
                <el-tab-pane label="法语" name="french" v-if="get_yuyan.includes('french')"> <el-input v-model="temp.desc5" /></el-tab-pane>
                <el-tab-pane label="德语" name="de" v-if="get_yuyan.includes('de')"> <el-input v-model="temp.desc6" /></el-tab-pane>
                <el-tab-pane label="西班牙语" name="xby" v-if="get_yuyan.includes('xby')"> <el-input v-model="temp.desc7" /></el-tab-pane>
                <el-tab-pane label="俄语" name="eyu" v-if="get_yuyan.includes('eyu')"> <el-input v-model="temp.desc8" /></el-tab-pane>
              </el-tabs>
            </el-form-item>
          </div>

          <el-form-item label="图片">
            <!-- <el-upload class="upload-demo" :on-preview="handlePreview" action="#" :on-remove="handleRemove"
              :on-change="get_img" :limit="1" ref="fileupload1" :file-list="fileList" list-type="picture">
              <el-button size="small" type="primary">选择图片</el-button>
              <div slot="tip" class="el-upload__tip">
                只能上传jpg/png文件，且不超过500kb
              </div>
            </el-upload> -->
            <el-upload action="#" list-type="picture-card" :on-change="get_img" :limit="1" ref="fileupload1"
              :auto-upload="false" style="margin-bottom:20px;width: 100%;">
              <ul class="el-upload-list el-upload-list--picture-card" v-show="isedit">
                <li tabindex="0" class="el-upload-list__item is-ready">
                  <div><img :src="card_img" alt="" class="el-upload-list__item-thumbnail">
                    <span class="el-upload-list__item-actions"><span class="el-upload-list__item-preview"><i
                          class="el-icon-plus" @click="isedit = false"></i></span></span>
                  </div>
                </li>
              </ul>
              <i slot="default" class="el-icon-plus" v-show="!isedit"></i>
              <div slot="file" slot-scope="{file}" v-show="!isedit">
                <img class="el-upload-list__item-thumbnail" :src="card_img" alt="" />
                <span class="el-upload-list__item-actions">
                  <span class="el-upload-list__item-preview" @click="handlePictureCardPreview(1, file)">
                    <i class="el-icon-zoom-in"></i>
                  </span>
                  <span v-if="!disabled" class="el-upload-list__item-delete" @click="handleRemoveimg(1, file)">
                    <i class="el-icon-delete"></i>
                  </span>
                </span>
              </div>
            </el-upload>
            <el-dialog :visible.sync="dialogVisible" z-index="999">
              <img width="100%" :src="dialogImageUrl" alt="">
            </el-dialog>
          </el-form-item>
        </div>
        <!-- <el-form-item prop="content" style="margin-bottom: 30px">
         
          <wang-editor v-model="temp.content" />
        </el-form-item> -->

        <el-form-item label="内容" prop="content" style="margin-bottom: 30px; margin-left: 50px;">
          <el-tabs v-model="activeName" type="border-card">

            <el-tab-pane label="繁体中文" name="zh"> <wang-editor v-model="temp.content" /></el-tab-pane>
            <el-tab-pane label="简体中文" name="zhCN"> <wang-editor v-model="temp.content1" /></el-tab-pane>
            <el-tab-pane label="英文" name="en"> <wang-editor v-model="temp.content2" /></el-tab-pane>
            <el-tab-pane label="日文" name="japan"> <wang-editor v-model="temp.content3" /></el-tab-pane>
            <el-tab-pane label="韩文" name="han"> <wang-editor v-model="temp.content4" /></el-tab-pane>
            <el-tab-pane label="法语" name="french"> <wang-editor v-model="temp.content5" /></el-tab-pane>
            <el-tab-pane label="德语" name="de"> <wang-editor v-model="temp.content6" /></el-tab-pane>
            <el-tab-pane label="西班牙语" name="xby"> <wang-editor v-model="temp.content7" /></el-tab-pane>
            <el-tab-pane label="俄语" name="eyu"> <wang-editor v-model="temp.content8" /></el-tab-pane>
          </el-tabs>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          {{ $t("table.cancel") }}
        </el-button>
        <el-button type="primary" @click="dialogStatus === 'create' ? createData() : updateData()">
          {{ $t("table.confirm") }}
        </el-button>
      </div>
    </el-dialog>
    <el-dialog title="图片详情" :visible.sync="see_img_dialog" center>
      <el-image :src="dialog_img"> </el-image>
    </el-dialog>
    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">{{
          $t("table.confirm")
        }}</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {
  fetchList,
  fetchPv,
  createArticle,
  updateArticle,
} from "@/api/article";
import { fetchnoticelist, createnotice, delnotice, home_zhu333 } from "@/api/room_sys";
import waves from "@/directive/waves"; // waves directive
import { parseTime } from "@/utils";
import Pagination from "@/components/Pagination"; // secondary package based on el-pagination
import Tinymce from "@/components/Tinymce";
const calendarTypeOptions = [];
import WangEditor from '@/components/WangEditor'
// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name;
  return acc;
}, {});

export default {
  name: "ComplexTable",
  components: { Pagination, Tinymce, WangEditor },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: "success",
        draft: "info",
        deleted: "danger",
      };
      return statusMap[status];
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type];
    },
  },
  data() {
    return {
      get_yuyan: [],
      activeName: 'zh',
      isedit: false,
      disabled: false,
      card_img: '',
      dialogImageUrl: '',
      dialogVisible: false,
      tableKey: 0,
      dialog_img: "",
      see_img_dialog: false,
      if_sel_img: false,
      list: [],
      total: 0,
      listLoading: false,
      listQuery: {
        page: 1,
        pagesize: 10,
      },
      importanceOptions: [1, 2, 3],
      calendarTypeOptions,
      sortOptions: [
        { label: "ID Ascending", key: "+id" },
        { label: "ID Descending", key: "-id" },
      ],

      showReviewer: false,
      temp: {
        id: undefined,
        title: "",
        title1: "",
        title2: "",
        title3: "",
        title4: "",
        title5: "",
        title6: "",
        title7: "",
        title8: "",

        type_id: 1,
        status: 1,
        img: "",

        desc: '',
        desc1: '',
        desc2: '',
        desc3: '',
        desc4: '',
        desc5: '',
        desc6: '',
        desc7: '',
        desc8: '',
        content: "",
        content1: "",
        content2: "",
        content3: "",
        content4: "",
        content5: "",
        content6: "",
        content7: "",
        content8: "",

      },
      dialogFormVisible: false,
      dialogStatus: "",
      textMap: {
        update: "Edit",
        create: "Create",
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        title: [{ required: true, message: "标题为必填项", trigger: "blur" }],
      },
      downloadLoading: false,
      multipleSelection: [],
      fileList: [],
    };
  },
  created() {
    this.getList();
    var get_yuyan = localStorage.getItem("language")
    this.get_yuyan = get_yuyan.split(',')
    
  },
  methods: {
    get_img(file, fileList) {
      this.if_sel_img = true
      this.temp.img = file.raw
      this.card_img = file.url
      //this.fileList.push({ name: file.name, url: file.url })
      //   let formData = new FormData();
      //  formData.append("file", file.raw);

    },
    handleRemoveimg(type, file) {
      this.temp.img = ''
      let uploadFiles
      if (type == 1) {
        uploadFiles = this.$refs['fileupload1'].uploadFiles;
      } else if (type == 2) {
        uploadFiles = this.$refs['fileupload2'].uploadFiles;
      } else if (type == 3) {
        uploadFiles = this.$refs['fileupload3'].uploadFiles;
      } else if (type == 4) {
        uploadFiles = this.$refs['fileupload4'].uploadFiles;
      }
      let index = uploadFiles.indexOf(file);
      uploadFiles.splice(index, 1);
    },
    handlePictureCardPreview(type, file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    handleSelection(val) {
      this.multipleSelection = val;
    },
    see_img(url) {

      this.see_img_dialog = true;
      this.dialog_img = url;
    },
    getList() {
      this.listLoading = true;
      fetchnoticelist(this.listQuery).then((response) => {
        if (response.code == 200) {
          this.list = response.data.list;
          this.total = response.data.total;
        }
        this.listLoading = false;
        // Just to simulate the time of the request
      });
    },

    handleModifyStatus(row, status) {
      this.$message({
        message: "操作成功",
        type: "success",
      });
      row.status = status;
    },
    sortChange(data) {
      const { prop, order } = data;
      if (prop === "id") {
        this.sortByID(order);
      }
    },

    resetTemp() {
      this.temp = {
        id: undefined,
        title: "",
        title1: "",
        title2: "",
        title3: "",
        title4: "",
        title5: "",
        title6: "",
        title7: "",
        title8: "",
        type_id: 1,
        status: 1,
        img: "",
        desc: '',
        desc1: '',
        desc2: '',
        desc3: '',
        desc4: '',
        desc5: '',
        desc6: '',
        desc7: '',
        desc8: '',
        content: "",
        content1: "",
        content2: "",
        content3: "",
        content4: "",
        content5: "",
        content6: "",
        content7: "",
        content8: "",

      };
    },
    handleCreate() {
      this.resetTemp();
      this.dialogStatus = "create";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    del_row(del_id) {
      this.$confirm("确定要删除该数据记录?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          delnotice({ id: del_id })
            .then((response) => {
              if (response.code == 200) {
                this.$message({
                  message: "删除成功",
                  type: "success",
                });
              }
              this.getList();
            })
            .catch((err) => { });
        })
        .catch(() => { });
    },
    edit_row(edit_data) {
      this.isedit = true
      this.card_img = edit_data.img;
      this.temp.id = edit_data.id;
      this.temp.title = edit_data.title;

      this.temp.title1 = edit_data.title_zhCN !==null?edit_data.title_zhCN:"";
      this.temp.title2 = edit_data.title_en !==null?edit_data.title_en:"";
      this.temp.title3 = edit_data.title_japan !==null?edit_data.title_japan:"";
      this.temp.title4 = edit_data.title_han !==null?edit_data.title_han:"";
      this.temp.title5 = edit_data.title_french !==null?edit_data.title_french:"";
      this.temp.title6 = edit_data.title_de !==null?edit_data.title_de:"";
      this.temp.title7 = edit_data.title_xby !==null?edit_data.title_xby:"";
      this.temp.title8 = edit_data.title_eyu !==null?edit_data.title_eyu:"";


      this.temp.type_id = edit_data.type;
      this.temp.status = edit_data.status;


      // this.temp.img = edit_data.img;
      this.temp.desc = edit_data.desc;
      this.temp.desc1 = edit_data.desc_zhCN !==null?edit_data.desc_zhCN:"";
      this.temp.desc2 = edit_data.desc_en !==null?edit_data.desc_en:"";
      this.temp.desc3 = edit_data.desc_japan !==null?edit_data.desc_japan:"";
      this.temp.desc4 = edit_data.desc_han !==null?edit_data.desc_han:"";
      this.temp.desc5 = edit_data.desc_french !==null?edit_data.desc_french:"";
      this.temp.desc6 = edit_data.desc_de !==null?edit_data.desc_de:"";
      this.temp.desc7 = edit_data.desc_xby !==null?edit_data.desc_xby:"";
      this.temp.desc8 = edit_data.desc_eyu !==null?edit_data.desc_eyu:"";

      this.temp.content =edit_data.content !==null? this.$utils.unescape(edit_data.content):"";

      this.temp.content1 =edit_data.content_zhCN !==null? this.$utils.unescape(edit_data.content_zhCN):"";
      this.temp.content2 =edit_data.content_en !==null? this.$utils.unescape(edit_data.content_en):"";
      this.temp.content3 =edit_data.content_japan !==null? this.$utils.unescape(edit_data.content_japan):"";
      this.temp.content4 =edit_data.content_han !==null? this.$utils.unescape(edit_data.content_han):"";
      this.temp.content5 =edit_data.content_french !==null? this.$utils.unescape(edit_data.content_french):"";
      this.temp.content6 =edit_data.content_de !==null? this.$utils.unescape(edit_data.content_de):"";
      this.temp.content7 =edit_data.content_xby !==null? this.$utils.unescape(edit_data.content_xby):"";
      this.temp.content8 =edit_data.content_eyu !==null? this.$utils.unescape(edit_data.content_eyu):"";

      this.dialogFormVisible = true;
    },
    createData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          let formData = new FormData();
          if (this.if_sel_img == true) {
            formData.append("img", this.temp.img);
          } else {
            formData.append("img", '');
          }

          formData.append("title", this.temp.title);
          formData.append("title_zhCN", this.temp.title1);
          formData.append("title_en", this.temp.title2);
          formData.append("title_japan", this.temp.title3);
          formData.append("title_han", this.temp.title4);
          formData.append("title_french", this.temp.title5);
          formData.append("title_de", this.temp.title6);
          formData.append("title_xby", this.temp.title7);
          formData.append("title_eyu", this.temp.title8);
          formData.append("type_id", this.temp.type_id);

          formData.append("status", this.temp.status);
          formData.append("content", this.temp.content);
          formData.append("content_zhCN", this.temp.content1);
          formData.append("content_en", this.temp.content2);
          formData.append("content_japan", this.temp.content3);
          formData.append("content_han", this.temp.content4);
          formData.append("content_french", this.temp.content5);
          formData.append("content_de", this.temp.content6);
          formData.append("content_xby", this.temp.content7);
          formData.append("content_eyu", this.temp.content8);
          formData.append("desc", this.temp.desc);
          formData.append("desc_zhCN", this.temp.desc1);
          formData.append("desc_en", this.temp.desc2);
          formData.append("desc_japan", this.temp.desc3);
          formData.append("desc_han", this.temp.desc4);
          formData.append("desc_french", this.temp.desc5);
          formData.append("desc_de", this.temp.desc6);
          formData.append("desc_xby", this.temp.desc7);
          formData.append("desc_eyu", this.temp.desc8);

          createnotice(formData).then((res) => {
            if (res.code == 200) {
              this.$notify({
                title: "成功",
                message: "创建成功",
                type: "success",
                duration: 2000,
              });
            }
            this.activeName = 'zh'
            this.dialogFormVisible = false;
            this.$refs.fileupload1.clearFiles();
            this.if_sel_img = false

            this.getList()
          });
        }
      });
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row); // copy obj
      this.temp.timestamp = new Date(this.temp.timestamp);
      this.dialogStatus = "update";
      this.dialogFormVisible = true;
      this.$nextTick(() => {
        this.$refs["dataForm"].clearValidate();
      });
    },
    updateData() {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp);
          let formData = new FormData();
          if (this.if_sel_img == true) {
            formData.append("img", this.temp.img);
          } else {
            formData.append("img", '');
          }
          formData.append("id", this.temp.id);
         
          formData.append("type_id", this.temp.type_id);

          formData.append("status", this.temp.status);
          formData.append("title", this.temp.title);

          formData.append("title_zhCN", this.temp.title1);
          formData.append("title_en", this.temp.title2);
          formData.append("title_japan", this.temp.title3);
          formData.append("title_han", this.temp.title4);
          formData.append("title_french", this.temp.title5);
          formData.append("title_de", this.temp.title6);
          formData.append("title_xby", this.temp.title7);
          formData.append("title_eyu", this.temp.title8);

          

          formData.append("desc", this.temp.desc);
          formData.append("desc_zhCN", this.temp.desc1);
          formData.append("desc_en", this.temp.desc2);
          formData.append("desc_japan", this.temp.desc3);
          formData.append("desc_han", this.temp.desc4);
          formData.append("desc_french", this.temp.desc5);
          formData.append("desc_de", this.temp.desc6);
          formData.append("desc_xby", this.temp.desc7);
          formData.append("desc_eyu", this.temp.desc8);
          formData.append("content", this.temp.content);
          formData.append("content_zhCN", this.temp.content1);
          formData.append("content_en", this.temp.content2);
          formData.append("content_japan", this.temp.content3);
          formData.append("content_han", this.temp.content4);
          formData.append("content_french", this.temp.content5);
          formData.append("content_de", this.temp.content6);
          formData.append("content_xby", this.temp.content7);
          formData.append("content_eyu", this.temp.content8);

          createnotice(formData).then((res) => {
            this.dialogFormVisible = false;
            if (res.code == 200) {
              this.$notify({
                title: "成功",
                message: "更新成功",
                type: "success",
                duration: 2000,
              });
            }
            this.activeName = 'zh'
            this.$refs.fileupload1.clearFiles();
            this.if_sel_img = false
            this.getList()
          });
        }
      });
    },
    handleDelete(row, index) {
      this.$notify({
        title: "成功",
        message: "删除成功",
        type: "success",
        duration: 2000,
      });
      this.list.splice(index, 1);
    },
    handleFetchPv(pv) {
      fetchPv(pv).then((response) => {
        this.pvData = response.data.pvData;
        this.dialogPvVisible = true;
      });
    },

    formatJson(filterVal) {
      return this.list.map((v) =>
        filterVal.map((j) => {
          if (j === "timestamp") {
            return parseTime(v[j]);
          } else {
            return v[j];
          }
        })
      );
    },
    getSortClass: function (key) {
      const sort = this.listQuery.sort;
      return sort === `+${key}` ? "ascending" : "descending";
    },
  },
};
</script>
<style></style>
