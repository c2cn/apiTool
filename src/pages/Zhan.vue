<template>
  <div>
    <!--头部-->
    <el-row class="head">
      <el-col :span="24" class="navigate">
        <el-menu default-active="1" mode="horizontal" theme="dark" menu-trigger="click" class="menu">
          <template v-for="(className, key) in classNameArr">
            <el-submenu :index="className" class="menu-submenu">
              <template slot="title" class="menu-submenu-title">{{className}}</template>
              <template v-for="(apiName, key2) in getApiArrByClassName(className)">
                <el-menu-item :index="apiName" class="submenu-item">
                  <div size="mini" @click="clickApi(className, apiName)" class="submenu-item-title">{{apiName}}</div>
                </el-menu-item>
              </template>
            </el-submenu>
          </template>
        </el-menu>
      </el-col>
    </el-row>
    <!--中间-->
    <el-row :gutter="10">
      <!--侧边api列表-->
      <el-col class="apiList" :span="4">
        <el-menu default-active="1" theme="dark">
          <template v-for="(className, key) in classNameArr">
            <el-submenu :index="className" class="menu-submenu">
              <template slot="title" class="menu-submenu-title">{{className}}</template>
              <template v-for="(apiName, key2) in getApiArrByClassName(className)">
                <el-menu-item :index="apiName" class="submenu-item">
                  <div size="mini" @click="clickApi(className, apiName)" class="submenu-item-title">{{apiName}}</div>
                </el-menu-item>
              </template>
            </el-submenu>
          </template>
        </el-menu>
      </el-col>
      <!--api操作-->
      <el-col :span="14" class="main">
        <el-row class="row-api-info">
          <el-card>
            <el-tag>api:</el-tag>
            <el-tag v-show="!api">请选择api</el-tag>
            <el-tag v-text="api"  v-show="api"></el-tag>
            <el-tooltip effect="dark" content="加入常用接口" placement="right">
              <el-tag v-show="api" type="success">
                <el-button size="mini" type="text" @click="recordCommonUseApi" icon="plus"></el-button>
              </el-tag>
            </el-tooltip>
          </el-card>
        </el-row>
        <!--输入-->
        <el-row class="row-api-input" v-show="api">
          <el-card>
            <div slot="header" class="clearfix">
              <el-tag>输入参数</el-tag>
            </div>
            <el-form label-position="top" onkeydown="if(event.keyCode==13){return false;}">
              <template v-for="(param, index) in params">
                <el-form-item :label="index + ': ' + param">
                  <el-input v-model="form[index]"></el-input>
                </el-form-item>
              </template>
              <el-form-item>
                <el-row type="flex" justify="end">
                  <el-col :span="24">
                    <el-button type="primary" @click="onSubmit" native-type="button">提交</el-button>
                  </el-col>
                </el-row>
              </el-form-item>
            </el-form>
          </el-card>
        </el-row>
        <el-row v-show="api">
          <el-card>
            <el-upload
              :action="uploadUrl"
              :on-preview="uploadFilePreview"
              :on-success="uploadFileSuccess"
              :on-error="uploadFileError"
              :data="uploadData"
              :headers="uploadHeaders"
            >
              <el-button size="small" type="primary">上传文件</el-button>
            </el-upload>
          </el-card>
        </el-row>
        <!--输出-->
        <el-row  class="row-api-output" v-show="api">
          <el-card>
            <div slot="header" class="clearfix">
              <el-tag v-text="">返回结果:</el-tag>
              <el-tag>服务器耗时:{{apiDoTime}}ms</el-tag>
              <el-tag>总耗时:{{allDoTime}}ms</el-tag>
              <el-tag v-show="retIsUploadFile" type="success">uploadRet</el-tag>
            </div>
            <el-tree :data="treeResult" :props="defaultProps"></el-tree>
          </el-card>
        </el-row>
      </el-col>
      <!--侧边栏-->
      <el-col :span="6">
        <!--服务器信息-->
        <el-row>
          <el-card>
              <el-tooltip content="服务器地址" placement="left">
                <el-tag v-text="server" hit type="success"></el-tag>
              </el-tooltip>
              <el-tooltip content="修改api服务器地址" placement="bottom">
                <el-button size="small" type="primary" @click.native="inputServer">修改</el-button>
              </el-tooltip>
            <el-tooltip content="刷新api列表" placement="bottom">
              <el-button size="small" type="primary" @click.native="refreshServer">刷新</el-button>
            </el-tooltip>
            <el-tooltip content="是否为hprose调用接口" placement="bottom">
              <el-switch
                v-model="isHprose"
                on-color="#13ce66"
                off-color="#ff4949">
              </el-switch>
            </el-tooltip>

          </el-card>
          <!--userId 显示-->
          <el-card class="loginInfo">
            <el-tooltip content="userId">
              <el-tag>{{userId}}</el-tag>
            </el-tooltip>
            <el-tooltip content="type">
              <el-tag>{{type}}</el-tag>
            </el-tooltip>
          </el-card>
        </el-row>
        <!--常用api-->
        <el-row>
          <el-card>
            <div slot="header">
              <span>常用接口</span>
            </div>
            <template v-for="(item, key) in getCommonUseList" style="display:table;">
              <div>
                <el-button-group>
                  <el-button type="small" @click="recordCommonUseApi(item.className, item.apiName)">{{item.use}}</el-button>
                  <el-button  icon="close" type="small"
                              @click="removeCommonUseApi(item.className, item.apiName)"></el-button>
                  <el-button  type="small"
                             @click="clickCommonApi(item)">{{item.className + '_' + item.apiName}}</el-button>
                </el-button-group>
              </div>
            </template>
          </el-card>
        </el-row>
        <!--常用服务器url-->
        <el-row>
          <el-card>
            <div slot="header">
              <span>常用服务器</span>
            </div>
            <template v-for="(item, key) in getCommonServerUrl" style="display:table;">
              <div>
                <el-button-group>
                  <el-button type="small" @click="recordCommonServerUrl(item.url)">{{item.use}}</el-button>
                  <el-button  icon="close" type="small"
                              @click="removeCommonServerUrl(item.url)"></el-button>
                  <el-button  type="small"
                              @click="changeServer(item.url)" v-text="item.url"></el-button>
                  <el-button  icon="edit" type="small"
                              @click="openRemarkBox(item.url)"></el-button>
                </el-button-group>
              </div>
            </template>
          </el-card>
        </el-row>
      </el-col>
    </el-row>
    <!--尾部-->
    <el-row type="flex" justify="end">
    </el-row>
  </div>
</template>
<script type="text/ecmascript-6">
// import hprose from '../static/hprose/hprose-html5'
// import {client} from '../common/hprose/hprose-vue'
import {toString} from '../common/filter/util'
let _ = require('lodash')
// let $ = require('jQuery')
import hprose from '../static/hprose/hprose-html5'
import * as cookie from '../common/cookie'
import * as apiControl from '../common/request'
// import * as apiControl from '../common/api-control'
let request = require('request')

export default {
  name: 'zhan',
  data () {
    return {
      // hprose对象
      client: {},
      defaultServer: 'http://192.168.2.67:8080',
      useServer: '',    // 正在使用的服务器地址
      classNameArr: [],
      apiNameArr: [],
      selectApi: [],     // 选中的api，用于界面显示
      // 执行api实际使用时间 毫秒
      allDoTime: null,
      startTime: null,
      endTime: null,
      // 表单数据
      form: {},
      // api返回数据
      result: {},
      // 登陆凭证 {once: {token: '', userId: ''}
      loginInfo: {},
      // 树形图使用数据
      treeResult: [],
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      // 常用api列表
      commonUseApiList: {},    // {"server": {"Tool_ip": {className: "Tool", apiName: "ip", use: 1}, ...}}
      commonUseServerUrl: {}, // 常用服务器地址 {'server': {'url': 'http://**.com', 'use': 1, 'remark': '可以编辑的记录'}}
      retIsUploadFile: false,
      fileInfoArr: [],
      // 上传文件时，附带的头信息
      uploadHeaders: {
//        'content-type': 'application/json'
      },
      // 新的数据存储
      apiList: {},    // 接口列表
      isHprose: true
    }
  },
  mounted () {
    // 取出cookie
    let loginInfo = cookie.getLoginInfo()
    let useServer = cookie.getUseServer()
    let commonUseApiList = cookie.getCommonUseApiList()
    let commonUseServerUrl = cookie.getCommonUseServerUrl()
    if (useServer) this.useServer = useServer
    if (loginInfo) this.loginInfo = loginInfo
    if (commonUseApiList) this.commonUseApiList = commonUseApiList
    if (commonUseServerUrl) this.commonUseServerUrl = commonUseServerUrl
    this.initData()
  },
  methods: {
    // 根据类名，获取该类下的api列表
    getApiArrByClassName (className) {
      return _.keys(_.get(this.apiNameArr, className))
    },
    // 点击api名称，切换api
    clickApi (className, apiName) {
      console.log('click : ', className + '_' + apiName)
      this.cleanData()
      this.selectApi = {'className': className, 'apiName': apiName}
      let form = {}
      let params = this.params
      for (let key of _.keys(params)) {
        if (params[key] === 'string') {
          form[key] = ''
        } else if (params[key] === 'array') {
          form[key] = ''
        } else if (params[key] === 'int' || params[key] === 'integer') {
          form[key] = ''
        } else {
          form[key] = ''
        }
      }
      this.form = form
    },
    clickCommonApi (item) {
      let className = item.className
      let apiName = item.apiName
      let form = item.form
      this.clickApi(className, apiName)
      if (!_.isEmpty(form)) {
        this.form = form
      }
    },
    // 提交表单
    onSubmit () {
      console.group('Submit to ' + this.useServer)
      console.log('send form data: ', this.form)
      if (!this.api) {
        console.error('未选择接口')
        return ''
      }
      this.retIsUploadFile = false
      this.startTime = (new Date()).getTime()
      // 发送请求
      let questUrl = this.useServer + '/' + this.api
      let form = this.handleArgs()
      if (!this.isHprose) {
        apiControl.post(questUrl, form, (error, response, body) => {
          if (error) {
            console.error('接口' + this.api + '返回错误: ', error)
          }
          this.parseSubmitReturn(body)
        })
      } else {
        let client = this.getClient()
        client.invoke(this.api, [form]).then((result) => {
          this.parseSubmitReturn(result)
        }).catch((error) => {
          console.error('接口' + this.api + '返回错误: ', error)
        })
      }
    },
    parseSubmitReturn (result) {
      // 处理接口返回值
      this.result = result
      this.treeResult = this.handleResultToTree(result)
      this.recordCommonUseApi()   // 调用记录
      // 计算接口调用使用时间 =>毫秒
      this.endTime = (new Date()).getTime()
      this.allDoTime = this.endTime - this.startTime
      // 记录登陆凭证
      if (_.has(result, 'user')) {
        console.info('接口' + this.api + '登陆成功,userId :', result.user.userId)
        this.loginInfo = result.user
        cookie.saveLoginInfo(result.user)
      }
      if (_.has(result, 'ret.code')) {
        if (result.ret.code === 'ok') {
          console.info('接口' + this.api + '返回成功数据 :', result)
          this.$message({
            type: 'success',
            message: '操作执行成功'
          })
        } else {
          console.error('接口' + this.api + '返回失败数据 :', result)
          console.error('error code: ' + result.ret.code)
          console.error('error msg: ' + result.ret.msg)
          this.$message({
            type: 'error',
            message: result.ret.msg
          })
        }
      }
      console.groupEnd()
    },
    getClient () {
      if (_.isEmpty(this.client)) {
        this.client = hprose.Client.create(this.useServer)
      }
      return this.client
    },
    // 处理传入接口的参数
    handleArgs () {
      let args = []
      let form = this.form
      _.forEach(form, (value, key) => {
        if (this.form[key]) {
          args.push(JSON.parse(value))
        } else {
          args.push('')
        }
      })
      let loginInfo = this.loginInfo
      let userId = _.get(loginInfo, 'userId', '')
      let token = _.get(loginInfo, 'token', '')
      let type = _.get(loginInfo, 'type', '')
      if (this.isHprose) {
        return {data: args, userId: userId, token: token, type: type}
      } else {
        return {data: args, isHttp: true, userId: userId, token: token, type: type}
      }
    },
    // 处理upload提交的数据
    handlePostArgs () {
      let userId = _.get(this.loginInfo, 'userId', '')
      let token = _.get(this.loginInfo, 'token', '')
      let argsObj = {userId: userId, token: token}
      // todo 这里组件不支持post字符串，导致无法传参数给服务器
//      let args = JSON.stringify(argsObj)
      return argsObj
    },
    cleanData () {
      this.selectApi = []
      this.result = {}
      this.treeResult = []
      this.form = {}
      this.allDoTime = null
      this.retIsUploadFile = false
    },
    initApiList () {
      let getApiListUrl = this.useServer + '/Test_getApiList'
      console.info(getApiListUrl)
      apiControl.post(getApiListUrl, {}, (error, response, body) => {
        if (error) {
          console.error('获取api数据失败')
          console.error(error)
          this.$message({type: 'error', showClose: true, message: '操作未生效, 服务器地址异常，无法加载api信息, 地址为：' + this.server})
          return false
        }
//        let apiList = JSON.parse(body)
        let apiList = body
        console.info('获取到接口信息', apiList)
        if (_.has(apiList, 'debug')) {
          delete apiList.debug
        }
        if (_.has(apiList, 'ret')) {
          delete apiList.ret
        }
        this.apiList = apiList
        this.apiNameArr = apiList
        this.classNameArr = _.keys(apiList)
        this.$message({type: 'success', showClose: true, message: '连接服务器成功：' + this.useServer})
        this.cleanData()
        this.recordCommonServerUrl(this.useServer) // 记录使用过的服务器链接
        cookie.saveUseServer(this.useServer)
      })
    },
    initData (server = '') {
      let useServer = ''
      if (server) {
        useServer = server
      } else {
        useServer = this.server
      }
      this.useServer = useServer
      this.initApiList()
    },
    inputServer () {
      this.$prompt('请输入服务器地址', '提示', {
      }).then(({ value }) => {
        this.changeServer(value)    // 修改服务器地址
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '取消输入'
        })
      })
    },
    changeServer (url) {
      let that = this
      request(url + '/Test_getApiList', function (error, response, body) {
        if (error) {
          console.error('无效的服务器地址', url, error)
          that.$message({
            type: 'error',
            showClose: true,
            message: '操作未生效，无效的服务器地址: ' + url
          })
        } else {
          that.initData(url)
        }
      })
    },
    // 处理返回数据，处理成树形空间使用格式
    handleResultToTree (data) {
      let result = []
      if (_.isObject(data)) {
        for (let key of _.keys(data)) {
          if (_.isObject(data[key])) {
            result.push({
              label: key,
              children: this.handleResultToTree(data[key])
            })
          } else {
            result.push({
              label: key + ': ' + data[key]
            })
          }
        }
      } else {
        result.push({
          label: 'undefined' + ': ' + data
        })
      }
      return result
    },
    // 记录使用的api数据
    recordCommonUseApi (className, apiName) {
      let apiInfoObj = this.selectApi
      if (className && apiName) {
        apiInfoObj = {
          className: className,
          apiName: apiName,
          form: this.form
        }
      }
      let fullApiName = apiInfoObj['className'] + '_' + apiInfoObj['apiName']   // 完整api名称
      let server = _.kebabCase(this.server)
      if (_.has(this.commonUseApiList, server + '.' + fullApiName)) {
        let newUse = _.get(this.commonUseApiList, server + '.' + fullApiName + '.use') + 1
        this.commonUseApiList = _.set(_.cloneDeep(this.commonUseApiList), server + '.' + fullApiName + '.' + 'use', newUse)
      } else {
        let initData = {
          className: apiInfoObj['className'],
          apiName: apiInfoObj['apiName'],
          use: 1,
          server: server,
          form: this.form
        }
        this.commonUseApiList = _.set(_.cloneDeep(this.commonUseApiList), server + '.' + fullApiName, initData)
      }
    },
    removeCommonUseApi (className, apiName) {
      let fullName = className + '_' + apiName
      this.commonUseApiList = _.omit(this.commonUseApiList, _.kebabCase(this.server) + '.' + fullName)
    },
    recordCommonServerUrl (url) {
      let server = _.kebabCase(url)
      if (_.has(this.commonUseServerUrl, server)) {
        let newUse = _.get(this.commonUseServerUrl, server + '.use') + 1
        this.commonUseServerUrl = _.set(_.cloneDeep(this.commonUseServerUrl), server + '.' + 'use', newUse)
      } else {
        let initData = {
          use: 1,
          url: url,
          remark: ''
        }
        this.commonUseServerUrl = _.set(_.cloneDeep(this.commonUseServerUrl), server, initData)
      }
    },
    removeCommonServerUrl (url) {
      this.commonUseServerUrl = _.omit(this.commonUseServerUrl, _.kebabCase(url))
    },
    remarkCommonServerUrl (url, remark) {
      let server = _.kebabCase(url)
      if (_.has(this.commonUseServerUrl, server)) {
        this.commonUseServerUrl = _.set(_.cloneDeep(this.commonUseServerUrl), server + '.' + 'remark', remark)
      } else {
        let initData = {
          use: 1,
          url: server,
          remark: remark
        }
        this.commonUseServerUrl = _.set(_.cloneDeep(this.commonUseServerUrl), server, initData)
      }
    },
    openRemarkBox (url) {
      this.$prompt('输入备注', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      }).then(({ value }) => {
        this.remarkCommonServerUrl(url, value)
        this.$message({
          type: 'success',
          message: '修改成功'
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '取消输入'
        })
      })
    },
    refreshServer () {
      if (!this.useServer) return false
      this.initData()
    },
    // upload file
    // 点击上传的文件时触发
    uploadFilePreview (file) {
      console.group('文件信息:' + file.name)
      console.info('uid:' + file.uid)
      console.info('服务器返回response:', _.get(this.fileInfoArr, file.uid, {}))
      console.info('file:', file)
      console.groupEnd()
    },
    // 上传文件成功
    uploadFileSuccess (response, file, fileList) {
      this.$message({
        type: 'success',
        message: '上传文件成功:' + file.name
      })
      console.group('上传文件成功:', file.name)
      console.info('服务器返回response:', response)
      console.info('file:', file)
      console.info('fileList:', fileList)
      console.groupEnd()
      _.set(this.fileInfoArr, file.uid, response)
      this.treeResult = this.handleResultToTree(response)
      this.retIsUploadFile = true   // 表示返回的数据是上传文件返回的
    },
    // 上传文件失败
    uploadFileError (err, response, file) {
      this.$message({
        type: 'error',
        message: '上传文件失败:' + file.name
      })
      console.group('上传文件失败:', file.name)
      console.error('err:', err)
      console.info('服务器返回response:', response)
      console.info('file:', file)
      console.groupEnd()
    }
  },
  computed: {
    api () {
      if (_.isEmpty(this.selectApi)) {
        return ''
      } else {
        return this.selectApi['className'] + '_' + this.selectApi['apiName']
      }
    },
    className () {
      if (_.isEmpty(this.selectApi)) {
        return ''
      } else {
        return this.selectApi['className']
      }
    },
    apiName () {
      if (_.isEmpty(this.selectApi)) {
        return ''
      } else {
        return this.selectApi['apiName']
      }
    },
    params () {
      return _.get(this.apiNameArr, [this.className, this.apiName])
    },
    // api执行时间
    apiDoTime () {
      if (_.has(this.result, 'debug.call.time')) {
        return this.result.debug.call.time
      } else {
        return null
      }
    },
    getCommonUseList () {
      let name = _.kebabCase(this.server)
      let apiList = _.get(this.commonUseApiList, name, {})
      return _.sortBy(apiList, function (n) {
        return -n['use']
      })
    },
    getCommonServerUrl () {
      return _.sortBy(this.commonUseServerUrl, function (n) {
        return -n['use']
      })
    },
    // 服务器地址  缓存 = 设置 > 默认
    server () {
      if (this.useServer) {
        return this.useServer
      } else {
        return this.defaultServer
      }
    },
    // 获取上传文件的接口链接
    uploadUrl () {
      return this.useServer + '/' + this.api
    },
    uploadData () {
      let data
      data = this.handlePostArgs()
      return data
    },
    userId () {
      return _.get(this.loginInfo, 'userId', '') + ''
    },
    token () {
      return _.get(this.loginInfo, 'token', '') + ''
    },
    type () {
      return _.get(this.loginInfo, 'type', '') + ''
    }
  },
  filters: {
    toString
  },
  watch: {
    commonUseApiList: function (val, oldVal) {
      cookie.saveCommonUseApiList(val)
    },
    commonUseServerUrl: function (val, oldVal) {
      cookie.saveCommonUseServerUrl(val)
    }
  }
}
</script>
<style scoped>
  .navigate {
  }
  .list {
  }
  .main {
  }
  .status {
  }
  .all {
    font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
  }
  .head {
    background-color: #1F2D3D;
    margin-bottom: 10px;
  }
  .row-api-info {
    margin-bottom: 10px;
  }
  .row-api-input {
    margin-bottom: 10px;
  }
  .row-api-output {
    margin-bottom: 10px;
  }
  .head .el-menu--horizontal .el-submenu {
    height: 40px;
    line-height: 40px;
  }
  .head .el-menu--horizontal .el-submenu .el-menu-item {
     height: 24px;
     padding:0 5px;
     line-height: 24px;
  }
  .loginInfo {
    font-size: 2px;
  }



</style>
