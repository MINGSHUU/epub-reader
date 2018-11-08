<template>
    <transition name="slide-right">
        <div class="content">
            <div class="content-wrapper" v-if="bookAvailable">
                <div class="content-item"
                     v-for="(item, index) in navigation.toc"
                     :key="index" @click="jumpTo(item.href)">
                    <span class="text">{{item.label}}</span>
                </div>
            </div>
            <div class="empty" v-else>加载中...</div>
        </div>
    </transition>
</template>

<script>
  export default {
    props: {
      ifShowContent: Boolean,
      navigation: Object,
      bookAvailable: Boolean
    },
    methods: {
      jumpTo(target) {
        this.$emit('jumpTo' ,target)
      }
    }
  }
</script>

<style lang="scss" scoped>
    @import '../assets/styles/global';
    .content{
        position: absolute;
        top: 0;
        left: 0;
        z-index: 103;
        width: 70%;
        height: 100%;
        background: #fff;
        .content-item {
            @include left;
            box-sizing: border-box;
            padding: px2rem(5) px2rem(10);
            font-size: px2rem(14);
            width: 100%;
            height: px2rem(80);
            border-bottom: px2rem(1) solid #ccc;
        }
        .empty {
            @include center;
            font-size: px2rem(16);
            width: 100%;
            height: 100%;
        }
    }
</style>