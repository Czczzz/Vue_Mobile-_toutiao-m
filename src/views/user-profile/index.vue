<template>
  <div class="user-profile">
    <!-- 导航栏 -->
    <van-nav-bar
      class="page-nav-bar"
      title="个人信息"
      left-arrow
      @click-left="$router.back()"
    />
    <!-- /导航栏 -->
    <van-cell title="头像" is-link>
      <van-image class="avatar" fit="cover" round :src="user.photo" />
    </van-cell>
    <van-cell
      title="昵称"
      :value="user.name"
      is-link
      @click="isUpdateNameShow = true"
    />
    <van-cell title="性别" :value="user.gender ? '女' : '男'" is-link />
    <van-cell title="生日" :value="user.birthdy || '1977-00-00'" is-link />
    <!-- 编辑昵称 -->
    <van-popup
      v-model="isUpdateNameShow"
      style="height: 100%;"
      position="bottom"
    >
      <update-name
        v-if="isUpdateNameShow"
        v-model="user.name"
        @close="isUpdateNameShow = false"
      />
    </van-popup>
    <!-- /编辑昵称 -->
  </div>
</template>
<script>
import { getUserProfile } from '@/api/user.js'
import UpdateName from './components/update-name'
export default {
  name: 'UserProfile',
  components: {
    UpdateName
  },

  data() {
    return {
      user: {}, // 个人信息
      isUpdateNameShow: false
    }
  },

  created() {
    this.loadProfile()
  },

  methods: {
    async loadProfile() {
      try {
        const res = await getUserProfile()
        this.user = res.data.data
      } catch (err) {
        this.$toast('获取数据失败')
      }
    }
  }
}
</script>

<style scoped lang="less">
.user-profile {
  .avatar {
    width: 60px;
    height: 60px;
  }
  .van-popup {
    background-color: #f5f7f9;
  }
}
</style>
