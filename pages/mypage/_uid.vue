<template>
  <div id="mypage">
    <div class="main">
      <div class="history-box">
        <h2 class="history-headder">
          購入履歴
        </h2>
        <client-only placeholder="Loading...">
          <ul class="history-cards">
            <li v-for="(history,index) in getItems" :key="index" class="cards">
              <img :src=history.url alt="product-img">
              <div class="info-box">
                <div class="history-num">注文番号:{{ history.uuid}}</div>
                <div class="history-inner-box">
                  <div class="product-name">{{history.name}}</div>
                  <div class="right-box">
                    <div class="history-date">{{history.orderedDate}}</div>
                    <div class="product-price">合計{{history.amount}}個</div>
                    <div class="product-price">¥{{history.price}}(税込)</div>
                  </div>
                </div>
                <div class="detail-link" @click="toDetail(history.product_id)">もう一度購入する</div>
              </div>
            </li>
          </ul>
          <paginate v-if="(getPageCount > 1)"
            :page-count="getPageCount"
            :page-range="3"
            :margin-pages="2"
            :click-handler="clickCallback"
            :prev-text="'前へ'"
            :next-text="'次へ'"
            :container-class="'pagination-box'"
            :page-class="'pagination-item'"
            :page-link-class="'pagination-item__link'"
            :prev-class="'pagination-btn pagination-prev'"
            :prev-link-class="'pagination-btn__link'"
            :next-class="'pagination-btn pagination-next'"
            :next-link-class="'pagination-btn__link'"
            :hide-prev-next="true"
            >
          </paginate>
        </client-only>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  middleware:["redirect"],
  data() {
    return {
      filterdHistories:[],
      historyData:[],
      parPage: 8,
      currentPage: 1,
    };
  },
  methods: {
    async getHistories() {
      // const historyData =await this.$axios.get("http://127.0.0.1:8000/api/history")
      const historyData =await this.$axios.get("https://ecback.kyosuke-k.com/api/history")
      this.historyData = historyData.data.data; 
      const sorttedHistoryData = this.historyData.sort(function(a,b) {
        if(a.created_at > b.created_at) {
          return -1;
        } else {
          return 1;
        }
      })
      const mappedHistories = sorttedHistoryData.map(value => 
        value.products.map(function(product) {
          const date = new Date(value.created_at);
          const mm = date.getMonth()+ 1 + "月"
          const orderedDate = date.getFullYear() + "年" + mm + date.getDate() + "日";
          const data = {
            "uuid":value.uuid,
            "orderedDate":orderedDate,
            "user_id":value.user_id,
            "product_id":product.id,
            "name":product.name,
            "price":product.price,
            "header":product.header,
            "description":product.description,
            "url":product.url,
            "amount":product.pivot.amount,
          }
          return data
        })
      )
      const flattedHistories = mappedHistories.flat()
      const currentUserId = this.$auth.user.id;
      this.filterdHistories = flattedHistories.filter(function(value) {
        if (value.user_id === currentUserId) {
          return value
        }
      });
      // console.log(this.filterdHistories);
    },
    toDetail(id) {
      const extractedPro = this.filterdHistories.splice(
      this.filterdHistories.findIndex((obj) => obj.product_id === id ),1);
      extractedPro[0].id = extractedPro[0].product_id
      // console.log(extractedPro);
      this.$store.dispatch('select/select', extractedPro)
      this.$router.push('detail')
    },
    clickCallback(pageNum) {
      this.currentPage = Number(pageNum);
    }
  },
  computed: {
    getItems() {
      const current = this.currentPage * this.parPage;
      const start = current - this.parPage;
      return this.filterdHistories.slice(start, current);
      },
    getPageCount() {
      return Math.ceil(this.filterdHistories.length / this.parPage);
    }
  },
  mounted() {
    this.getHistories();
  }
}
</script>
<style scoped>
.main {
  margin:0 auto;
  width: 90%;
  min-height: 100vh;
  padding-top: 70px;
}
.history-box {
  width: 63%;
  margin: 0 auto;
}
.history-headder {
  font-weight: bold;
  font-size: 30px;
  line-height: 43px;
  border-bottom: 1px solid #828282;
  padding-bottom: 30px;
}
.cards {
  display: flex;
  border-bottom: 1px solid #828282;
  padding: 20px 0;
}
.cards img {
  min-width: 150px;
  max-width: 150px;
  height: 150px;
  object-fit: cover;
}
.history-cards {
  padding: 0 !important;
  margin-bottom: 40px;
}
.info-box {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-left: 20px;
  font-size: 14px;
  line-height: 19px;
  color: #6C6C6C;
  width: 100%;
  
}
.history-inner-box {
  width: 100%;
  display: flex;
  justify-content: space-between;
}
.product-name {
  font-size: 25px;
  line-height: 26px;
  color: #000000;
}
.history-inner-box .right-box {
  width: 50%;
  display: flex;
  justify-content: space-between;
  padding-right: 10px;
}
.detail-link {
  cursor: pointer;
}
</style>

