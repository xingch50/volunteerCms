<template>
  <div class="__pictures__">
    <!--<div class="formBox">-->
      <!--<el-form :inline="true" :model="formInline" class="demo-form-inline">-->
        <!--<el-form-item label="标题：">-->
          <!--<el-input v-model="title" placeholder="请输入标题"></el-input>-->
        <!--</el-form-item>-->
        <!--<el-form-item>-->
          <!--<el-button type="primary" icon="el-icon-search" @click="onSubmit">查询</el-button>-->
        <!--</el-form-item>-->
      <!--</el-form>-->
    <!--</div>-->
    <div class="btnBox">
      <el-button type="primary" icon="el-icon-circle-plus-outline" @click="onClickAdd">新增轮播图</el-button>
    </div>
    <div class="tableList">
      <vtable :dataArray="dataArray" :columns="columns" :total="total" @getArticle="queryUserListPost"></vtable>
    </div>
  </div>
</template>

<script>
  import MyDropDown from '@/components/common/MyDropDown';
  import table from '@/components/common/table';
  import {queryPictures} from "api/pictures/index";//列表
  import {upadtePictures} from "api/pictures/index";//修改
  export default {
    name: "pictures",
    components: {
      vtable: table
    },
    data() {
      return {
        title: '',
        formInline: {
          truename: '',
        },
        pageSize: 10,
        pageNum: 1,
        total: 0,
        dataArray: [],
        columns: [
          {
            prop: "status",
            label: "状态",
            render: function(createElement) {
              return createElement('img', {
                domProps: {
                  src: this.row.url,
                  className: 'imgWidth'
                }
              })
            },
          },
          {
            prop: 'title',
            label: '标题'
          },
          {
            prop: 'fUrl',
            label: '外链接'
          },
          {
            prop: 'updateTime',
            label: '最后修改时间'
          },
          {
            prop: '',
            label: '操作',
            render: (h, param) => {
              var items = [
                {label: "修改", func: {func: "update", uuid: param.row.uuid}},
                {label: "删除", func: {func: "del", uuid: param.row.uuid}},
              ]
              const dropDownData = {
                label: "操作",
                items: items
              };
              return h(MyDropDown, {
                props: {
                  dropDownData: dropDownData
                },
                on: {
                  update: this.update,
                  del: this.del
                }
              });
            }
          }
        ]
      }
    },
    created() {
      this.queryUserListPost(this.pageNum)
    },
    methods: {
      queryUserListPost(pageNum, title) {
        let params = {
          pageSize: this.pageSize,
          pageNum: pageNum,
          title: title,
          flag: 1
        }
        queryPictures(params).then(data => {
          if(data.data.status==200){
            this.total = data.data.total
            this.dataArray = data.data.data
          }
        })
      },
      onClickAdd() {
        this.$router.push({path: '/home/picturesAdd', query: { type: 1}})
      },
      del(uuid) {
        this.$confirm('是否删除该图片?', '温馨提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          upadtePictures({uuid: uuid, flag: 2}).then(data => {
            if(data.data.status==200){
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: '500',
                onClose: function(){
                  window.location.reload();
                }
              });
            }
          })
        }).catch(() => {
        });
      },
      update(uuid) {
        this.$router.push({path: '/home/picturesAdd', query: {uuid: uuid, type: 2}})
      }
    }
  }
</script>

<style lang="stylus">
  .__pictures__ {
    background #ffffff;
    padding 15px;
    box-shadow: 0 1px 2px 0 rgba(0,0,0,0.05);
    border-radius: 4px;
    .formBox {
      border-bottom 1px solid #eee
    }
    .btnBox {
      margin 10px 0;
    }
    .imgWidth {
      width 36px
      height 36px
    }
  }
</style>
