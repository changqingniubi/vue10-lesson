<template>
  <div>
    <MHeader>列表页</MHeader>
    <div class="content" ref="scroll" @scroll="loadMore">
      <ul>
        <router-link
          v-for="(book,index) in books"
          :to="{name:'detail',params:{bid:book.bookId}}"
          :key="index"
          tag="li"
        >
          <img v-lazy="book.bookCover">
          <div>
            <h4>{{book.bookName}}</h4>
            <p>{{book.bookInfo}}</p>
            <b>{{book.bookPrice}}</b>
            <div class="btn-list"> <button @click.stop="remove(book.bookId)">删除</button>
              <button @click.stop.prevent="addCart(book)">添加购物</button></div>

          </div>
        </router-link>
      </ul>
      <div @click="more" class="more">加载更多</div>
    </div>
  </div>
</template>
<script>
import {pagination,removeBook} from '../api';
import MHeader from '../base/MHeader.vue';
import {mapMutations} from 'vuex';
import * as Types from '../vuex/mutation-types'
export default {
    data(){
        // offset代表的是偏移量 hasMore 是否有更多  默认不是正在加载
        return {msg: 'hello',books:[],offset:0,hasMore:true,isLoading:false}
    },
    mounted(){
        let scroll = this.$refs.scroll; //获取到要拖拽的元素
        let top = scroll.offsetTop;
        let distance = 0;
        let m = false;
        scroll.addEventListener('touchstart',(e)=> {
            // 滚动条在最顶端时 并且当前盒子在顶端时候 才继续走
            if(scroll.scrollTop !=0 || scroll.offsetTop != top) return
          console.log(scroll.scrollTop,scroll.offsetTop)
          let start = e.touches[0].pageY; //手指点击的开始
            let move = (e)=>{
              m = true
              let current = e.touches[0].pageY;
              distance = current - start; //求的拉动的距离 负的就不要了
              if(distance>10){ // 如果大于50了 认为就是50像素
                if(distance<=50){
                  scroll.style.top = distance + top +'px';
                }else{
                  distance = 50;
                  scroll.style.top = top+50+'px';
                }
              }else{
                // 如果不在考虑范围内 移除掉move和end事件
                scroll.removeEventListener('touchmove',move);
                scroll.removeEventListener('touchend',end);
              }
            };
            let end = (e)=>{
                if(!m)return;
                m = false
              clearInterval(this.timer1);
              this.timer1 = setInterval(()=>{ // 一共distance 每次-1
                if(distance<=0){
                  clearInterval(this.timer1);
                  distance = 0;
                  scroll.style.top = top+'px';
                  scroll.removeEventListener('touchmove',move);
                  scroll.removeEventListener('touchend',end);
                  this.books = []; // 先清空数据
                  this.offset = 0;
                  this.hasMore = true;
                  this.getData();
                  return
                }
                distance -=1;
                scroll.style.top =top + distance+'px';
              },1);
              scroll.removeEventListener('touchmove',move);
              scroll.removeEventListener('touchend',end);
            };
          scroll.addEventListener('touchmove',move);
          scroll.addEventListener('touchend',end);
        },false);
    },
    created(){
      this.getData();
    },
    methods: {
        ...mapMutations([Types.ADD_CART]),
        addCart(book){
          this[Types.ADD_CART](book)
        },
        loadMore(){
            // 卷去的高度   当前可见区域  总高
          // 触发scroll事件 可能触发n次  先进来开一个定时器，下次触发时将上一次定时器清除掉
          clearTimeout(this.timer); // 节流
          this.timer = setTimeout(()=>{
            let {scrollTop,clientHeight,scrollHeight} = this.$refs.scroll;
            if(scrollTop+clientHeight+20>scrollHeight){
                this.getData(); // 获取更多
            }
          },60);
        },
        more(){
            this.getData();
        },
        async remove(id){
          await removeBook(id); // 删除某一项
          // 要删除前台数据
          this.books = this.books.filter(item=>item.bookId!==id);
        },
        async getData(){
            if(this.hasMore&&!this.isLoading){ // 有更多的时候获取数据
              this.isLoading = true;
              let {hasMore,books} = await pagination(this.offset);
              this.books = [...this.books,...books]; // 获取的书放到books属性上
              this.hasMore = hasMore;
              this.isLoading = false; // 加载完毕
              this.offset = this.books.length; //维护偏移量 就是总书的长度
            }

        }
    },
    computed: {},
    components: {MHeader}
}
</script>
<style scoped>
  .content ul{
    padding: 10px;
  }
  .content ul li{
    display: flex;
    padding: 10px 0;
    border-bottom: 1px solid #f1f1f1;
  }
  .content ul li img{
    width:130px;
    height: 150px;
  }
  .content h4{
    font-size: 20px;
    line-height: 35px;
  }
  .content p{
    color: #2a2a2a;
    line-height: 25px;
  }
  .content b {
    color: red;
  }
  .content button{
    display: block;width: 60px;height: 25px; background: orangered; color: #fff;border: none; border-radius: 15px; outline: none;
  }
  .more{
    margin: 10px;
    background: #2afedd;
    height: 30px;
    line-height: 30px;
    text-align: center;
    font-size: 20px;
  }
  .btn-list{
    display: flex;
    justify-content: space-around;
  }

</style>
