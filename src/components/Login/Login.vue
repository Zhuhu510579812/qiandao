<template>
  <div>
    <div class="login">
      <el-form :model="form" :label-position="labelPosition" ref="form" :rules="rules" label-width="100px">
        <el-form-item label="手机号码：" prop='phoneNum'>
          <el-input v-model.number="form.phoneNum" type="text" placeholder="手机号码">

          </el-input>
        </el-form-item>

        <el-form-item label="用户密码：" prop='passWord'>
          <el-input v-model="form.passWord" type="password" placeholder="请输入密码">
          </el-input>
        </el-form-item>

        <el-form-item label="用户身份：" prop="identity">
          <el-radio-group v-model="form.identity">
            <el-radio class="radio" label="领导"></el-radio>
            <el-radio class="radio" label="教师"></el-radio>
            <el-radio class="radio" label="学生"></el-radio>
          </el-radio-group>
        </el-form-item>

      </el-form>
      <el-button type="primary" @click="Login('form')" size="large" style="width:160px">登录</el-button>
      <el-button type="primary" @click="reset('form')" size="large" style="width:160px">取消</el-button>
    </div>
  </div>
</template>

<script>
import { setCookie, getCookie } from '../../common/js/cookie'
export default {
  mounted() {
    // 页面挂载获取cookie 存在则跳转
    if (getCookie('username')) {
      this.$router.push('/leader')
    }
  },
  data() {
    var checkPhoneNum = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('手机号码不能为空'))
      }
      setTimeout(() => {
        if (!Number.isInteger(value)) {
          callback(new Error('请输入数字类型值'))
        } else {
          if (value.toString().length !== 11) {
            callback(new Error('手机号码长度错误'))
          } else {
            callback()
          }
        }
      }, 1000)
    }
    return {
      labelPosition: 'right',
      form: {
        phoneNum: '',
        passWord: '',
        identity: ''
      },
      rules: {
        phoneNum: [
          {
            validator: checkPhoneNum,
            trigger: 'blur'
          }
        ],
        passWord: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          }
        ],
        identity: [
          {
            required: true,
            message: '请选择登录身份',
            trigger: 'change'
          }
        ]
      }
    }
  },
  methods: {
    Login(form) {
      var identity = this.form.identity
      let data = {
        'num': this.form.phoneNum,
        'pass': this.form.passWord
      }
      this.$refs[form].validate((valid) => {
        if (valid) {
          if (identity === '领导') {
            this.$axios.post('/api/user/leaderLogin', data).then((response) => {
              console.log(response)
              if (response.data) {
                setCookie('leader', this.form.phoneNum, 1000 * 60)
                setTimeout(function() {
                  this.$router.push({
                    path: `/leader`
                  })
                }.bind(this), 500)
              } else {
                this.err()
              }
            })
          } else if (identity === '教师') {
            this.$axios.post('/api/user/teacherLogin', data).then((response) => {
              console.log(response)
              // console.log(obj.num)
              if (response.data.length === 1) {
                localStorage.setItem('teacher_info', response.data[0].tid)
                setCookie('teacher', this.form.phoneNum, 1000 * 60)
                setTimeout(function() {
                  this.$router.push({
                    path: `/teacher`
                  })
                }.bind(this), 500)
              } else {
                this.err()
              }
            })
          } else if (identity === '学生') {
            this.$axios.post('/api/user/studentLogin', data).then((response) => {
              const obj = JSON.parse(response.config.data)
              if (obj) {
                setCookie('studentno', obj.num, 1000 * 60)
                setCookie('student', this.form.phoneNum, 1000 * 60)
                setTimeout(function() {
                  this.$router.push({
                    path: `/student`
                  })
                }.bind(this), 500)
              } else {
                this.err()
              }
            })
          }
          // console.log('submit!')
        } else {
          // console.log('error submit!!')
          return false
        }
      })
    },
    reset(form) {
      this.$refs[form].resetFields()
    },
    err() {
      this.$message('用户名不存在或密码错误')
    }
  },
  components: {
  }
}
</script>


<style scoped lang="stylus" rel="stylesheet/stylus">
.login
  margin:0 auto;
  width:400px;
  padding:75px 0px;
  text-align :center
</style>
