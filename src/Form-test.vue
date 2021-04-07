<template>
  <wang-form class="form" ref="form" :model="user" :rules="rules">
    <wang-form-item label="用户名" prop="username">
      <!-- <wang-input v-model="user.username"></wang-input> -->
      <wang-input :value="user.username" @input="user.username=$event" placeholder="请输入用户名"></wang-input>
    </wang-form-item>
    <wang-form-item label="密码" prop="password">
      <wang-input type="password" v-model="user.password"></wang-input>
    </wang-form-item>
    <wang-form-item>
      <wang-button type="primary" @click="login">登 录</wang-button>
    </wang-form-item>
  </wang-form>
</template>

<script>
import WangForm from './form/Form'
import WangFormItem from './form/FormItem'
import WangInput from './form/Input'
import WangButton from './form/Button'
export default {
  components: {
    WangForm,
    WangFormItem,
    WangInput,
    WangButton
  },
  data () {
    return {
      user: {
        username: '',
        password: ''
      },
      rules: {
        username: [
          {
            required: true,
            message: '请输入用户名'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码'
          },
          {
            min: 6,
            max: 12,
            message: '请输入6-12位密码'
          }
        ]
      }
    }
  },
  methods: {
    login () {
      this.$refs.form.validate(valid => {
        if (valid) {
          alert('验证成功')
        } else {
          alert('验证失败')
          return false
        }
      })
    }
  }
}
</script>

<style>
  .form {
    width: 30%;
    margin: 150px auto;
  }
</style>
