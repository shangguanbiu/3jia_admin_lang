<template>
  <div class="app-container">
    <div class="filter-container">
      <el-button class="filter-item" style="margin-left: 10px;" icon="el-icon-back" @click="handleBack">
        返回
      </el-button>
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        {{ $t('table.add') }}
      </el-button>

    </div>

    <el-table :key="tableKey" v-loading="listLoading" :data="list" stripe fit highlight-current-row style="width: 100%;"
      @sort-change="sortChange">

      <el-table-column label="名称" min-width="120" prop="name">
      </el-table-column>

      <el-table-column label="代码" min-width="120" prop="code">
      </el-table-column>
      <el-table-column label="是否启用" min-width="80" prop="statue">
        <template slot-scope="scope">
          <span style="color:#53d6c4;cursor: pointer;" v-if="scope.row.status == 1">启用</span>
          <span style="color:#ff643a;cursor: pointer;" v-else>禁用</span>
        </template>
      </el-table-column>
      <el-table-column label="最低投注" min-width="120" prop="limtbit">
        <template slot-scope="scope">
          {{ formatNum(scope.row.limit_amount) }}
        </template>
      </el-table-column>
      <el-table-column label="最高投注" min-width="120" prop="limtbit">
        <template slot-scope="scope">
          {{ formatNum(scope.row.max_amount) }}
        </template>
      </el-table-column>
      <el-table-column label="最低入房金额限制" min-width="140" prop="limtmoney">
        <template slot-scope="scope">
          {{ formatNum(scope.row.limit_user_amount) }}
        </template>
      </el-table-column>
      <!-- <el-table-column label="玩法说明" min-width="140" prop="content">
        <template slot-scope="scope">
          <el-tooltip :content="scope.row.remark">
            <span v-if="scope.row.remark ? scope.row.remark.length > 5 : false"> {{
              scope.row.remark.slice(0, 4)
            }}... </span>
            <span v-else> {{ scope.row.remark }} </span>
          </el-tooltip>
        </template>
      </el-table-column> -->
      <el-table-column label="缩略图">
        <template slot-scope="scope">

          <el-image style="width: 50px; height: 50px" @click="see_img(scope.row.img)" :src="scope.row.img" fit="cover"
            lazy>
            <div slot="error" class="image-slot">
              <i class="el-icon-picture-outline" style="font-size:330%"></i>
            </div>
          </el-image>
        </template>
      </el-table-column>
      <el-table-column label="创建时间" min-width="150" prop="create_time">
      </el-table-column>
      <el-table-column label="操作" width="250">
        <template slot-scope="scope">
          <span style="color:#ff643a;cursor: pointer;" @click="del_row(scope.row.id)">删除</span> |
          <span style="color:#3E3EF7;cursor: pointer;" @click="handleUpdate(scope.row)">编辑</span> |
          <span style="color:#3E3EF7;cursor: pointer;" @click="room_peilv(scope.row)">设置赔率</span> |
          <span style="color:#53d6c4;cursor: pointer;" @click="build_jiangqi(scope.row.id)">生成奖期</span>

        </template>
      </el-table-column>
    </el-table>

    <pagination v-show="total > 0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.pagesize"
      @pagination="getList" />

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible"  width="90%">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="120px"
        style="width: 100%; margin-left:50px;">
        <div style="width: 500px;">
          <el-form-item label="代码" prop="code">
            <el-input v-model="temp.code" />
          </el-form-item>
        </div>
          <!-- <el-form-item label="房间名称" prop="name">
            <el-input v-model="temp.name" />
          </el-form-item> -->
          <div style="width: 90%;">
          <el-form-item label="房间名称" prop="name">
            <el-tabs v-model="activeName" type="border-card" id="zi_room">
              
              <el-tab-pane label="繁体中文" name="zh"> <el-input v-model="temp.name" id="001" @input="changeThis" /></el-tab-pane>
              <el-tab-pane label="简体中文" name="zhCN" v-if="get_yuyan.includes('zhCN')"> <el-input v-model="temp.name1" id="002" @input="changeThis" /></el-tab-pane>
              <el-tab-pane label="英文" name="en" v-if="get_yuyan.includes('en')"> <el-input v-model="temp.name2" id="003" @input="changeThis" /></el-tab-pane>
              <el-tab-pane label="日文" name="japan" v-if="get_yuyan.includes('japan')"> <el-input v-model="temp.name3" id="004" @input="changeThis" /></el-tab-pane>
              <el-tab-pane label="韩文" name="han" v-if="get_yuyan.includes('han')"> <el-input v-model="temp.name4" id="005"  @input="changeThis"/></el-tab-pane>
              <el-tab-pane label="法语" name="french" v-if="get_yuyan.includes('french')"> <el-input v-model="temp.name5" id="006"  @input="changeThis"/></el-tab-pane>
              <el-tab-pane label="德语" name="de" v-if="get_yuyan.includes('de')"> <el-input v-model="temp.name6" id="007"  @input="changeThis"/></el-tab-pane>
              <el-tab-pane label="西班牙语" name="xby" v-if="get_yuyan.includes('xby')"> <el-input v-model="temp.name7" id="008" @input="changeThis"/></el-tab-pane>
              <el-tab-pane label="俄语" name="eyu" v-if="get_yuyan.includes('eyu')"> <el-input v-model="temp.name8" id="009" @input="changeThis"/></el-tab-pane>
            </el-tabs>
          </el-form-item>
        </div>
          <div style="width: 500px;">
          <el-form-item label="最低下注" prop="limit_amount">
            <el-input v-model="temp.limit_amount" type="number" />
          </el-form-item>
          <el-form-item label="最高下注" prop="max_amount">
            <el-input v-model="temp.max_amount" type="number" />
          </el-form-item>
          <el-form-item label="入房最低金额" prop="limit_user_amount">
            <el-input v-model="temp.limit_user_amount" type="number" />
          </el-form-item>
          <el-form-item label="是否启用">
            <el-radio v-model="temp.status" :label="1">启用</el-radio>
            <el-radio v-model="temp.status" :label="0">禁用</el-radio>
          </el-form-item>
          <!-- <el-form-item label="交易说明">
            <el-input v-model="temp.remark" type="textarea" placeholder="请输入内容" />
          </el-form-item> -->
        </div>
        
        <el-form-item label="缩略图">
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
        <!-- <div style="width: 70%;">
          <el-form-item label="交易说明">
            
            <wang-editor v-model="temp.remark" />
          </el-form-item>
        </div> -->
        <el-form-item  label="游戏说明" prop="remark" style="margin-bottom: 30px;">
          <el-tabs v-model="activeName" type="border-card" style="width: 90%;">
            <el-tab-pane label="繁体中文" name="zh"> <wang-editor v-model="temp.remark" /></el-tab-pane>
            <el-tab-pane label="简体中文" name="zhCN"  v-if="get_yuyan.includes('zhCN')"> <wang-editor v-model="temp.remark1" /></el-tab-pane>
            <el-tab-pane label="英文" name="en"  v-if="get_yuyan.includes('en')"> <wang-editor v-model="temp.remark2" /></el-tab-pane>
            <el-tab-pane label="日文" name="japan"  v-if="get_yuyan.includes('japan')"> <wang-editor v-model="temp.remark3" /></el-tab-pane>
            <el-tab-pane label="韩文" name="han"  v-if="get_yuyan.includes('han')"> <wang-editor v-model="temp.remark4" /></el-tab-pane>
            <el-tab-pane label="法语" name="french"  v-if="get_yuyan.includes('french')"> <wang-editor v-model="temp.remark5" /></el-tab-pane>
            <el-tab-pane label="德语" name="de"  v-if="get_yuyan.includes('de')"> <wang-editor v-model="temp.remark6" /></el-tab-pane>
            <el-tab-pane label="西班牙语" name="xby"  v-if="get_yuyan.includes('xby')"> <wang-editor v-model="temp.remark7" /></el-tab-pane>
            <el-tab-pane label="俄语" name="eyu"  v-if="get_yuyan.includes('eyu')"> <wang-editor v-model="temp.remark8" /></el-tab-pane>
          </el-tabs>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          {{ $t('table.cancel') }}
        </el-button>
        <el-button type="primary" @click="dialogStatus === 'create' ? createData() : updateData()">
          {{ $t('table.confirm') }}
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">{{ $t('table.confirm') }}</el-button>
      </span>
    </el-dialog>
    <el-dialog title="图片详情" :visible.sync="see_img_dialog" center>
      <el-image :src="dialog_img">
      </el-image>
    </el-dialog>
  </div>
</template>
  
<script>
import { fetchList, fetchPv, createArticle, updateArticle } from '@/api/article'
import { fetchxiaoroomlist, addxiaoroom, delxiaoroom, build_jiangqi } from '@/api/room_sys'
import WangEditor from '@/components/WangEditor'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const calendarTypeOptions = [
  { key: 1, code: 'VIP1', display_name: '大房间1', img: '' },
  { key: 2, code: 10002, display_name: '欧元/美元' },
  { key: 3, code: 10003, display_name: '英镑/美元' },
  { key: 4, code: 10004, display_name: '美元日元' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'ComplexTable',
  components: { Pagination,WangEditor },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type]
    }
  },
  data() {
    return {
      activeName:'zh',
      isedit: false,
      disabled: false,
      card_img: '',
      dialogImageUrl: '',
      dialogVisible: false,
      tableKey: 0,
      dialog_img: "",
      see_img_dialog: false,
      dialog_img: '',
      see_img_dialog: false,
      tableKey: 0,
      list: [],
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        pagesize: 10,
        pid: null,

      },
      importanceOptions: [1, 2, 3],
      calendarTypeOptions,
      sortOptions: [{ label: 'ID Ascending', key: '+id' }, { label: 'ID Descending', key: '-id' }],

      showReviewer: false,
      temp: {
        id: undefined,
        room_id: '',
        code: '',
        name: '',
        name1: "",
        name2: "",
        name3: "",
        name4: "",
        name5: "",
        name6: "",
        name7: "",
        name8: "",
        limit_amount: '',
        limit_user_amount: '',
        max_amount: '',
        status: 1,
        remark: '',
        remark1: "",
        remark2: "",
        remark3: "",
        remark4: "",
        remark5: "",
        remark6: "",
        remark7: "",
        remark8: "",
        img: ''

      },
      id: '',
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      if_sel_img: false,
      dialogPvVisible: false,
      isedit: false,
      card_img: '',
      pvData: [],
      rules: {
        code: [{ required: true, message: '请输入代号编码', trigger: 'change' }],
        name: [{ required: true, message: '请输入房间名称', trigger: 'change' }],
        limit_amount: [{ required: true, message: '请输入最低下注金额', trigger: 'change' }],
        max_amount: [{ required: true, message: '请输入最高下注金额', trigger: 'change' }],
        limit_user_amount: [{ required: true, message: '请输入最低入房金额', trigger: 'change' }]
      },
      downloadLoading: false,
      get_yuyan: [],
    }
  },
  created() {
    this.listQuery.pid = Number(this.$route.query.id)
    this.temp.room_id = Number(this.$route.query.id)
    this.getList()
    var get_yuyan = localStorage.getItem("language")
    this.get_yuyan = get_yuyan.split(',')
  },
  methods: {
    formatNum(value) {
      if (!value && value !== 0) return 0;
      let str = value.toString();
      let reg =
        str.indexOf(".") > -1 ? /(\d)(?=(\d{3})+\.)/g : /(\d)(?=(?:\d{3})+$)/g;
      return str.replace(reg, "$1,");
    },
    handleBack() {
      this.$router.push({
        path: "/room/index", query: {}
      })
    },
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
    see_img(url) {
      this.see_img_dialog = true;
      this.dialog_img = url
    },
    getList() {
      this.listLoading = true
      fetchxiaoroomlist(this.listQuery).then(response => {
        if (response.code == 200) {
          this.list = response.data.list
          this.total = response.data.total
        }
        this.listLoading = false
      })
    },
    handleFilter() {
      this.listQuery.page = 1
      this.listQuery.pagesize = 1
      this.getList()
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: '操作成功',
        type: 'success'
      })
      row.status = status
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'id') {
        this.sortByID(order)
      }
    },
    sortByID(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+id'
      } else {
        this.listQuery.sort = '-id'
      }
      this.handleFilter()
    },
    changeThis() {
     this.$forceUpdate()
    
    },
    resetTemp() {
      this.temp = {

        code: '',
        name: '',
        limit_amount: '',
        limit_user_amount: '',
        max_amount: '',
        status: 1,
        remark: '',
        name1: "",
        name2: "",
        name3: "",
        name4: "",
        name5: "",
        name6: "",
        name7: "",
        name8: "",
        remark1: "",
        remark2: "",
        remark3: "",
        remark4: "",
        remark5: "",
        remark6: "",
        remark7: "",
        remark8: "",
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.room_id = this.listQuery.pid


          let formData = new FormData();
          if (this.if_sel_img == true) {
            formData.append("img", this.temp.img);
          } else {
            formData.append("img", '');
          }
          formData.append("room_id", this.temp.room_id);
          formData.append("code", this.temp.code);
          formData.append("name", this.temp.name);
          formData.append("name_zhCN", this.temp.name1);
          formData.append("name_en", this.temp.name2);
          formData.append("name_japan", this.temp.name3);
          formData.append("name_han", this.temp.name4);
          formData.append("name_french", this.temp.name5);
          formData.append("name_de", this.temp.name6);
          formData.append("name_xby", this.temp.name7);
          formData.append("name_eyu", this.temp.name8);
          formData.append("limit_amount", this.temp.limit_amount);
          formData.append("limit_user_amount", this.temp.limit_user_amount);
          formData.append("max_amount", this.temp.max_amount);
          formData.append("status", this.temp.status);
          formData.append("remark", this.temp.remark);
          formData.append("remark_zhCN", this.temp.remark1);
          formData.append("remark_en", this.temp.remark2);
          formData.append("remark_japan", this.temp.remark3);
          formData.append("remark_han", this.temp.remark4);
          formData.append("remark_french", this.temp.remark5);
          formData.append("remark_de", this.temp.remark6);
          formData.append("remark_xby", this.temp.remark7);
          formData.append("remark_eyu", this.temp.remark8);

          addxiaoroom(formData).then((res) => {
            if (res.code == 200) {
              this.$notify({
                title: '成功',
                message: '创建成功',
                type: 'success',
                duration: 2000
              })
            }
            this.activeName='zh'
            this.$refs.fileupload1.clearFiles();
            this.if_sel_img = false
            this.dialogFormVisible = false
            this.getList()
          })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.isedit = true
      this.card_img = row.img;
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.temp.remark = this.$utils.unescape(row.remark);

      this.temp.name1 = row.name_zhCN !==null?row.name_zhCN:"";
      this.temp.name2 = row.name_en !==null?row.name_en:"";
      this.temp.name3 = row.name_japan !==null?row.name_japan:"";
      this.temp.name4 = row.name_han !==null?row.name_han:"";
      this.temp.name5 = row.name_french !==null?row.name_french:"";
      this.temp.name6 = row.name_de !==null?row.name_de:"";
      this.temp.name7 = row.name_xby !==null?row.name_xby:"";
      this.temp.name8 = row.name_eyu !==null?row.name_eyu:"";

      this.temp.remark1 =row.remark_zhCN !==null? this.$utils.unescape(row.remark_zhCN):"";
      this.temp.remark2 =row.remark_en !==null? this.$utils.unescape(row.remark_en):"";
      this.temp.remark3 =row.remark_japan !==null? this.$utils.unescape(row.remark_japan):"";
      this.temp.remark4 =row.remark_han !==null? this.$utils.unescape(row.remark_han):"";
      this.temp.remark5 =row.remark_french !==null? this.$utils.unescape(row.remark_french):"";
      this.temp.remark6 =row.remark_de !==null? this.$utils.unescape(row.remark_de):"";
      this.temp.remark7 =row.remark_xby !==null? this.$utils.unescape(row.remark_xby):"";
      this.temp.remark8 =row.remark_eyu !==null? this.$utils.unescape(row.remark_eyu):"";




      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    del_row(del_id) {
      this.$confirm("确定要删除该数据记录?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          delxiaoroom({ id: del_id })
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
    build_jiangqi(sub_id) {
      build_jiangqi({ room_id: this.temp.room_id, room_sub_id: sub_id })
        .then((response) => {
          if (response.code == 200) {
            this.$message({
              message: "生成成功",
              type: "success",
            });
          }
          this.getList();
        })
        .catch((err) => { });
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.room_id = this.temp.pid
          const tempData = Object.assign({}, this.temp)

          let formData = new FormData();
          if (this.if_sel_img == true) {
            formData.append("img", this.temp.img);
          } else {
            formData.append("img", '');
          }
          formData.append("id", this.temp.id);
          formData.append("room_id", this.temp.room_id);
          formData.append("code", this.temp.code);
          formData.append("name", this.temp.name);
          formData.append("name_zhCN", this.temp.name1);
          formData.append("name_en", this.temp.name2);
          formData.append("name_japan", this.temp.name3);
          formData.append("name_han", this.temp.name4);
          formData.append("name_french", this.temp.name5);
          formData.append("name_de", this.temp.name6);
          formData.append("name_xby", this.temp.name7);
          formData.append("name_eyu", this.temp.name8);
          formData.append("limit_amount", this.temp.limit_amount);
          formData.append("limit_user_amount", this.temp.limit_user_amount);
          formData.append("max_amount", this.temp.max_amount);
          formData.append("status", this.temp.status);
          formData.append("remark", this.temp.remark);
          formData.append("remark_zhCN", this.temp.remark1);
          formData.append("remark_en", this.temp.remark2);
          formData.append("remark_japan", this.temp.remark3);
          formData.append("remark_han", this.temp.remark4);
          formData.append("remark_french", this.temp.remark5);
          formData.append("remark_de", this.temp.remark6);
          formData.append("remark_xby", this.temp.remark7);
          formData.append("remark_eyu", this.temp.remark8);
          formData.append("img", this.temp.img);

          addxiaoroom(formData).then((res) => {
            if (res.code == 200) {
              this.$notify({
                title: '成功',
                message: '更新成功',
                type: 'success',
                duration: 2000
              })
            }
            this.activeName='zh'
            this.$refs.fileupload1.clearFiles();
            this.if_sel_img = false
            this.dialogFormVisible = false
            this.getList()
          })
        }
      })
    },
    handleDelete(row, index) {
      this.$notify({
        title: '成功',
        message: '删除成功',
        type: 'success',
        duration: 2000
      })
      this.list.splice(index, 1)
    },
    room_peilv(data) {

      this.$router.push({
        path: "/room/peilvlist", query: { room_id: data.pid, room_sub_id: data.id }
      })
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['timestamp', 'title', 'type', 'importance', 'status']
        const filterVal = ['timestamp', 'title', 'type', 'importance', 'status']
        const data = this.formatJson(filterVal)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list'
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal) {
      return this.list.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    getSortClass: function (key) {
      const sort = this.listQuery.sort
      return sort === `+${key}` ? 'ascending' : 'descending'
    }
  }
}
</script>
  