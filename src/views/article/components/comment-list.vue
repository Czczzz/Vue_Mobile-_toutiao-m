<template>
  <!-- 评论列表 -->
  <!--
    只有 List 在可视范围内才会检查滚动位置触发 onLoad
   -->
  <!--  immediate-check  是否在初始化时立即执行滚动位置检查 -->
  <!-- 因为我们在created 调用了onLoad 要页面一加载就获取评论总数量, -->
  <!-- 然后 只有 List 在可视范围内才会检查滚动位置触发 onLoad ,后面打开评论列表它就会又触发一次,就会造成回复列表数据重复 -->
  <van-list
    v-model="loading"
    :finished="finished"
    finished-text="已显示全部评论"
    :error.sync="error"
    error-text="加载失败,请点击重试"
    :immediate-check="false"
    @load="onLoad"
  >
    <comment-item
      v-for="(item, index) in list"
      :key="index"
      :comment="item"
      @reply-click="$emit('reply-click', $event)"
    />
  </van-list>
</template>

<script>
import { getComments } from '@/api/comment'
import CommentItem from './comment-item.vue'
export default {
  components: { CommentItem },
  name: 'CommentList',
  props: {
    source: {
      type: [Number, String, Object],
      required: true
    },
    list: {
      type: Array,
      default: () => []
    },
    type: {
      type: String,
      // 自定义 Prop 数据验证
      validator(value) {
        return ['a', 'c'].includes(value)
      },
      default: 'a'
    }
  },
  data() {
    return {
      // list: [], // 评论列表
      loading: false, // 上拉加载更多的 loading
      finished: false, // 是否加载结束
      offset: null, // 获取下一页数据的标记
      limit: 10,
      error: false
    }
  },
  created() {
    // 当你手动初始 onLoad 的话，它不会自动开始初始的 loading
    // 页面一加载就可以看获取到评论总数量    list只有在可视范围内才会滚动位置触发onLOad事件 渲染 评论列表数据
    this.loading = true
    this.onLoad()
  },
  methods: {
    async onLoad() {
      try {
        // 获取文章的评论和获取评论的回复是同一个接口
        // 唯一的区别是接口参数不一样
        //    type
        //      a 文章的评论
        //      c 评论的回复
        //    source
        //      文章的评论，则传递文章的 ID
        //      评论的回复，则传递评论的 ID
        // 1. 请求获取数据
        const { data } = await getComments({
          type: this.type, //  评论类型，a-对文章(article)的评论，c-对评论(comment)的回复
          source: this.source.toString(), // 源id，文章id或评论id
          offset: this.offset, // 获取评论数据的偏移量，值为评论id，表示从此id的数据向后取，不传表示从第一页开始读取数据
          limit: this.limit // 获取的评论数据个数，不传表示采用后端服务设定的默认每页数据量
        })

        // 2. 将数据添加到列表中
        const { results } = data.data
        this.list.push(...results)

        // 把文章评论的总数量传递到外部
        this.$emit('onload-success', data.data)

        // 3. 将 loading 设置为 false
        this.loading = false

        // 4. 判断是否还有数据
        if (results.length) {
          // 有就更新获取下一页的数据页码
          this.offset = data.data.last_id
        } else {
          // 没有就将 finished 设置结束
          this.finished = true
        }
      } catch (err) {
        this.error = true
        this.loading = false
      }
    }
  }
}
</script>

<style scoped lang="less">
.publish-wrap {
  position: fixed;
  left: 0;
  bottom: 0;
  width: 100%;
}

.van-list {
  margin-bottom: 45px;
}
</style>
