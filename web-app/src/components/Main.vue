<template>
  <el-container class="main">
    <!--header -->
    <el-header class="main-top">
      <!-- 顶栏 -->
      <el-row  type="flex" justify="space-between">
        <el-col :span="2" style="text-align: center">基础框架</el-col>
        <el-col :span="4" class="main-dropdown">
          <img src="../assets/logo.png" class="main-avatar" />
          <el-dropdown @command="handleCommand">
            <span class="el-dropdown-link">
              下拉菜单<i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="a">个人信息</el-dropdown-item>
              <el-dropdown-item command="b">注销</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-col>
      </el-row>
    </el-header>
    <el-container>
      <!--左侧菜单 -->
      <el-aside class="main-left">
        <el-menu class="el-menu-vertical-demo" :unique-opened="true" :router="true" text-color="#fff" background-color="#545c64" active-text-color="#ffd04b">
          <main-menu v-for="item in menuInfo" :module="item" :key="item.id" @click="clickMenu"></main-menu>
        </el-menu>
      </el-aside>
      <el-container class="main-right">
        <!--面包屑、标签-->
        <el-header class="main-breadcrumxxb">
          <el-row>
            <el-col :span="24">
              <el-breadcrumb separator="/" class="main-breadcrumb">
                <el-breadcrumb-item v-for="item in breadcrumb" :to="item.path ? { path: item.path } : null" :key="item.path">
                  {{ item.title }}
                </el-breadcrumb-item>
              </el-breadcrumb>
              <div class="main-tags">
                <el-tag size="medium" :key="index" v-for="(tag, index) in tags" @close="handleClose(index,tag)" closable :type="tag.active == false ? 'info' : null" style="margin-right: 10px;">
                  <span @click="clickTag(tag)">{{tag.title}}</span>
                </el-tag>
              </div>
            </el-col>
          </el-row>
        </el-header>
        <!--content -->
        <el-main class="main-content">
          <router-view></router-view>
        </el-main>
        <!--footer-->
        <el-footer height="40px">
          <el-col :span="24" class="main-foot">
            2018 - 2019 © fp2952
          </el-col>
        </el-footer>
      </el-container>
    </el-container>
  </el-container>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'

var Menu = {
  template: '<el-menu-item v-if="module.subModules.length === 0" :index="module.modulePath" ' +
  ':title="module.moduleName" :parent="module.parentId" :key="module.id" @click="click">' +
  '<i :class="module.moduleIcon ? module.moduleIcon : \'el-icon-star-off\'"></i>' +
  '<span slot="title">{{ module.moduleName }}</span>' +
  '</el-menu-item>' +
  '<el-submenu v-else-if="module.subModules.length > 0" :title="module.moduleName" :parent="module.parentId" ' +
  ':index="module.modulePath" :key="module.id">' +
  '<template slot="title">' +
  '<i :class="module.moduleIcon ? module.moduleIcon : \'el-icon-star-off\'"></i>' +
  '<span slot="title">{{ module.moduleName }}</span>' +
  '</template>' +
  '<main-menu v-for="item in module.subModules" :module="item" :key="item.id" @click="click"></main-menu>' +
  '</el-submenu>',
  name: 'main-menu',
  props: ['module'],
  methods: {
    click (menu) {
      this.$emit('click', menu)
    }
  }
}

export default {
  components: {
    'main-menu': Menu
  },
  data () {
    return {
      tags: [],
      breadcrumb: [{ title: '首页', path: '/' }]
    }
  },
  created () {
    this.geRoleInfo()
  },
  methods: {
    ...mapActions([
      'geRoleInfo'
    ]),
    clickMenu (menu, parent) {
      // 点击事件调用
      if (!parent) {
        // 清空面包屑
        this.breadcrumb = []
        this.breadcrumb.push({ title: menu.$attrs.title, path: menu.index })
        this.resetTags(menu.index)
        // 添加标签
        if (!this.checkTags(menu.index)) {
          // 如果标签总数已到达最大值，则删除第一个标签再进行添加
          if (this.tags.length === this.tagLenght) {
            this.tags.splice(0, 1)
          }
          this.tags.push({ title: menu.$attrs.title, path: menu.index, active: true })
        }
      } else {
        this.breadcrumb.push({ title: menu.$attrs.title, path: null })
      }
      if (menu.$attrs.parent) {
        this.clickMenu(menu.$parent.$parent, true)
      } else {
        // 递归完毕
        this.breadcrumb.push({ title: '首页', path: '/' })
        // 反向数组
        this.breadcrumb.reverse()
      }
    },
    resetTags (path) {
      this.tags.forEach(item => {
        if (item.path === path) {
          item.active = true
        } else {
          item.active = false
        }
      })
    },
    checkTags (path) {
      // 检查是否已存在
      for (var i = 0; i < this.tags.length; i++) {
        if (this.tags[i].path === path) {
          return true
        }
      }
      return false
    },
    handleCommand (command) {
      console.log(command)
    },
    clickTag (tag) {
      // 点击跳转到指定页面
      this.resetTags(tag.path)
      this.$router.push({ path: tag.path })
    },
    handleClose (index, tag) {
      // 如果已经没有标签页
      if (this.tags.length === 1) {
        this.$router.push({ path: '/' })
        this.tags.splice(index, 1)
        return
      }
      // 如果不是当前选中的，直接删除
      if (!tag.active) {
        this.tags.splice(index, 1)
        return
      }
      // 如果不是最后一个，关闭后则选中下一个标签页
      if (this.tags.length - 1 !== index) {
        this.tags[index + 1].active = true
        this.$router.push({ path: this.tags[index + 1].path })
      } else if (this.tags.length - 1 === index) {
        // 如果是最后一个，则选中上一个
        this.tags[index - 1].active = true
        this.$router.push({ path: this.tags[index - 1].path })
      }
      this.tags.splice(index, 1)
    }
  },
  computed: {
    ...mapGetters([
      'userInfo',
      'menuInfo',
      'mainLoading'
    ])
  }
}
</script>

<style>
  .main-top {
    padding: 15px;
    color: #fff;
    background-color: #414a52;
  }
  .main {
    height:100%;
    display: flex;
    background: #f5f7f9;
  }
  .main-left {
    width: 210px !important;
  }
  .main-right {
    width: 100%;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }
  .main-content {
    background-color: #fff;
    margin: 15px;
    border-radius: 2px;
    flex-grow: 1;
    padding: 10px;
  }
  .main-foot {
    display: flex;
    justify-content: center;
    align-items: center;
    color: #9ea7b4;
    font-size: 12px;
    line-height: 1.5;
    margin-bottom: 15px;
  }
  .main-dropdown {
    display: flex;
    justify-content: flex-end;
    align-items: center;
  }
  .main-avatar {
    width: 25px;
    height: 25px;
    border-radius: 25px;
    border: 1px solid #ccc;
    margin-right: 5px;
  }
  .main-breadcrumxxb {
    height: 40px !important;
  }
  .main-breadcrumb {
    padding-top: 20px;
    float: left;
  }
  .main-tags {
    padding-top: 12px;
    margin-left: 20px;
    float: left;
  }
  .el-menu-vertical-demo {
    height: 100%;
  }
  .el-menu-vertical-demo:not(.el-menu--collapse) {
    width: 200px;
  }
  .el-dropdown-link {
    cursor: pointer;
    color: #fff;
  }
</style>
