<template>
  <div class="m-wrapper m-wbox">
    <div class="m-pos-f m-box-model m-main">
      <header class="m-box m-justify-bet m-aln-center m-main m-bb1 m-head-top">
        <div class="m-box m-flex-grow1 m-aln-center m-flex-base0">
          <a href="javascript:;" @click="goBack">取消</a>
        </div>
        <div class="m-box-model m-flex-grow1 m-aln-center m-flex-base0 m-head-top-title">
          <span>发布动态</span>
        </div>
        <div class="m-box m-flex-grow1 m-aln-center m-flex-base0 m-justify-end">
          <svg v-if="loading" class="m-style-svg m-svg-def rotate">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#base-loading"></use>
          </svg>
          <a 
            v-else
            class="m-send-btn"
            :class="{ disabled }"
            @click="beforePost"
            href="javascript:;">发布</a>
        </div>
      </header>
      <main
         class="m-reles-con m-lim-width m-box-model m-flex-shrink1 m-flex-grow1"
         @click.self='areaFocus'>
         <content-text
         :rows='10'
         class='m-reles-txt-wrap'
         ref="contentText" />
      </main>
       <footer class="m-box-model m-flex-shrink1 m-flex-grow1 m-aln-center">
         <v-switch
         class="m-box m-bt1 m-bb1 m-lim-width m-pinned-row"
         type="checkbox"
         v-model="pinned">
           <slot>是否收费</slot>
         </v-switch>
         <div 
         class="m-box-model m-lim-width"
         :style="{visibility: pinned ? '' : 'hidden'}">
           <div class="m-pinned-amount-btns">
              <p class="m-pinned-amount-label">设置文字收费金额</p>
              <div class="m-box m-aln-center ">              
                <button 
                  :key="item"
                  v-for="item in items"
                  class="m-pinned-amount-btn"
                  :style="{ width: `${1 / items.length * 100}%` }"
                  :class="{ active: ~~amount === ~~item }"
                  @click="chooseDefaultAmount(item)">{{ ~~item.toFixed(2) }}</button>
              </div>
           </div>
           <div class="m-box m-aln-center m-justify-bet m-bb1 m-bt1 m-pinned-row plr20 m-pinned-amount-customize">
             <span>自定义金额</span>
             <div class="m-box m-aln-center">
              <input type="number" v-model="customAmount" placeholder="输入金额" dir="rtl">
              <p>积分</p>
             </div>
           </div>
           <p class="m-pinned-amount-label plr20">注：超过{{limit}}字部分内容收费</p>
         </div>
       </footer>
    </div>
  </div>
</template>
<script>
import { mapGetters } from "vuex";

import ContentText from "./components/ContentText.vue";
export default {
  name: "post-text",
  components: {
    ContentText
  },
  data() {
    return {
      loading: false,
      contentText: "",
      curpos: 0,
      scrollHeight: 0,
      pinned: false,

      amount: 0,
      customAmount: null
    };
  },
  computed: {
    ...mapGetters(["compose"]),
    disabled() {
      return !(this.compose.length > 0);
    },
    items() {
      return this.$store.state.CONFIG.feed.items || [];
    },
    limit() {
      return this.$store.state.CONFIG.feed.limit || 50;
    }
  },
  watch: {
    customAmount(val) {
      val && (this.amount = ~~val);
    }
  },
  methods: {
    areaFocus() {
      this.$refs.contentText.areaFocus();
    },
    chooseDefaultAmount(amount) {
      this.customAmount && (this.customAmount = null);
      this.amount = amount;
    },
    beforePost() {
      this.pinned
        ? this.amount === 0
          ? this.$Message.error("请设置收费金额")
          : this.compose.length <= this.limit
            ? this.$Message.error(`正文内容不足${this.limit}字, 无法设置收费`)
            : this.postText()
        : this.postText();
    },
    postText() {
      this.$http
        .post(
          "feeds",
          {
            feed_content: this.compose,
            feed_from: 2,
            feed_mark:
              new Date().valueOf() + "" + this.$store.state.CURRENTUSER.id
          },
          {
            validateStatus: s => s === 201
          }
        )
        .then(({ data }) => {
          this.$Message.success(data);
          this.goBack();
        })
        .catch(error => {
          console.log(error);
        });
    }
  }
};
</script>
<style>
.plr20 {
  padding-left: 20px;
  padding-right: 20px;
}
.m-pinned-row {
  font-size: 30px;
  height: 100px;
}
</style>
