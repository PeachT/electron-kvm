<template>
  <div class="wh100" ref="girder">
    <el-container class="wh100">
      <el-aside style="border-right:1px solid #EDF2FC;" width="224px">
        <menu-two :menuData="menuData" :nowName.sync="nowName" @add="add" @edit="edit" @down="down" @del="del" @save="save" @cancel="cancel"></menu-two>
      </el-aside>
      <el-main class="task-main">
        <h1 v-if="!nowData">没有数据</h1>
        <div v-if="nowData" class="wh100">
          <el-form :model="nowData" :rules="nowDataRules" ref="nowData" label-width="100px" v-if="editState">
            <el-form-item label="构件名称" prop="name">
              <el-input v-model="nowData.name"></el-input>
            </el-form-item>
          </el-form>
          <girder-info :holes.sync="nowData.holes"></girder-info>
        </div>
      </el-main>
    </el-container>
  </div>
</template>

<script>
  import girderInfo from './template/girderInfo';
  import MenuTwo from '../menus/menuTow';

  const baseData = {
    name: '',
    holes: [],
  };
  // hole: {
  //   name: '',
  //   detail: [], // 孔明细
  //   state: true, // 启用禁用
  // },
  export default {
    name: 'girder',
    components: {
      MenuTwo, girderInfo,
    },
    computed: {
      // 编辑状态
      editState() {
        return this.$store.state.global.editState;
      },
      addState() {
        return this.$store.state.global.addState;
      },
    },
    updated() {
      if (this.editState) {
        this.disabled(null);
      } else {
        this.disabled();
      }
    },
    beforeMount() {
      this.getMenuData();
      console.log(this.menuData);
      if (this.menuData.length > 0) {
        this.nowName = this.menuData[0].name;
      }
    },
    data: () => ({
      role: false,
      nowData: null,
      nowDataRules: {
        name: [
          { required: true, message: '请输入构件名称', trigger: 'blur' },
          { min: 1, max: 15, message: '长度在 1 到 15 个字符', trigger: 'blur' },
        ],
      },
      menuData: null,
      nowName: null,
    }),
    watch: {
      // 切换输入框状态
      editState(nval) {
        this.disabled(nval ? null : true);
      },
      // 切换菜单选项
      nowName(nval, oval) {
        if (nval !== null) {
          this.switchMenu();
        }
      },
    },
    methods: {
      // 菜单数据
      getMenuData() {
        try {
          const menus = window.girderDB.reverseGetAll().map((item) => {
            return {
              name: item.name,
            };
          });
          console.log('数据', menus);
          if (menus.length > 0) {
            if (this.nowName === null) {
              this.nowName = menus[0].name;
            }
          } else {
            this.nowData = null;
          }
          this.menuData = menus;
        } catch (error) {}
      },
      // 切换菜单
      switchMenu() {
        console.log('切换菜单', this.nowName);
        try {
          this.nowData = this.$unity.copyObj(window.girderDB.getOne({ name: this.nowName }));
        } catch (error) {
          this.errorShow(`${error}`);
        }
      },
      add() {
        this.$message('添加');
        this.nowName = null;
        this.nowData = this.$unity.copyObj(baseData);
      },
      edit() {
        this.$message('编辑');
      },
      down() {
        this.$message('下载');
      },
      del() {
        this.$confirm('您确定删除该成员吗？', '危险警告！', {
          confirmButtonText: '取消',
          cancelButtonText: '继续删除',
          type: 'warning',
        }).then(() => {
          this.$message({
            type: 'info',
            message: '取消删除！',
          });
        }).catch(() => {
          try {
            window.girderDB.del({ name: this.nowName });
            this.nowName = null;
            this.getMenuData();
            this.$message('删除成功！');
          } catch (error) {
            this.errorShow(`删除出错！${error}`);
          }
        });
      },
      save() {
        this.$message('保存');
        this.$refs.nowData.validate((valid) => {
          if (valid && !this.supervisorsEdit) {
            const collection = window.girderDB.c;
            const nowData = this.nowData;
            let msg = '添加成功！';
            let errorMsg = '数据插入出错！';
            try {
              // 添加
              if (this.addState) {
                // 判断用户名是否存在
                nowData.id = `${this.$unity.timeId()}T`;
                if (window.girderDB.insert(nowData, { name: nowData.name })) {
                  this.$message.error('名字重复！请重新输入！');
                  return;
                }
                this.nowName = nowData.name;
                // 修改
              } else {
                msg = '修改成功！';
                errorMsg = '数据更新出错！';
                window.girderDB.update(nowData);
              }
              this.$message.success(msg);
              this.getMenuData();
              this.$store.commit('editState', false);
              this.$store.commit('addState', false);
            } catch (error) {
              // this.$notify.error({
              //   showClose: true,
              //   duration: 0,
              //   title: '错误',
              //   message: `${errorMsg}--${error}`,
              // });
              this.errorShow(`${errorMsg}--${error}`);
            }
          } else {
            const msg = '输入有误！请安规输入！';
            this.$message.error(msg);
          }
        });
      },
      cancel() {
        const msg = this.addState ? '您确定要取消添加钢绞线规格吗？' : '您确定要放弃修改钢绞线规格吗？';
        this.$confirm(msg, '提示', {
          confirmButtonText: '取消编辑',
          cancelButtonText: '继续编辑',
          type: 'warning',
        }).then(() => {
          this.getMenuData();
          this.$store.commit('editState', false);
          this.$store.commit('addState', false);
          this.$message({
            type: 'info',
            message: '已成功取消编辑!',
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '继续编辑',
          });
        });
      },
      errorShow(msg) {
        this.$notify.error({
          showClose: true,
          duration: 0,
          title: '错误',
          message: msg,
        });
      },
      // 编辑禁止状态切换
      disabled(state = true) {
        const input = this.$refs.girder;
        this.$d3.select(input).selectAll('input').attr('disabled', state);
      },
    },
  };

</script>

