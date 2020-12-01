<template>
  <div class="new-address">
    <van-nav-bar
      :title="aid ? '编辑地址' : '新增地址'"
      left-text="返回"
      left-arrow
      :fixed="true"
      @click-left="back"
    />

    <van-address-edit 
      show-postal
      :show-delete="!!aid"
      show-set-default
      :area-columns-placeholder="['请选择', '请选择', '请选择']" 
      :area-list="areaList"
      :address-info="addressInfo"
      @save="saveAddress"
      @delete="removeAddress"
    />
  </div>
</template>

<script>
  import '../assets/less/newAddress.less'
  import areaList from '../assets/js/arealist'
  import {utils} from '../assets/js/utils'
  export default {
    data() {
      return {
        areaList,
        //地址数据初始值（收货人信息初始值）
        addressInfo: {
          id: '',
          name: '',
          tel: '',
          province: '',
          city: '',
          county: '',
          addressDetail: '',
          areaCode: '',
          postalCode: '',
          isDefault: false
        },

        aid: ''
      };
    },

    created() {
      //获取aid
      this.aid = this.$route.query.aid;
      

      // 编辑地址过来会有参数aid传过来的
      if (this.aid) {
        this.getAddressDataByAid();
      }
      
    },

    methods: {

      //返回上一级
      back() {
        this.$router.go(-1);
      },

      //新增地址
      newAddress(address) {

        let content = Object.assign({}, address);

        //获取token字符串
        let tokenString = localStorage.getItem('CSTK');
        if (!tokenString) {
          return this.$router.push({name: 'Login'});
        }
        
        delete content.id;
        delete content.country;

        /* 其中 isDefault 的值只能为 0 或者 1, ==> 0: 非默认地址, */
        content.isDefault = Number(content.isDefault);

        content.appkey = this.appkey;
        content.tokenString = tokenString

        //参数序列化
        let data = utils.queryString(content);

        

        this.$toast.loading({
          message: '加载中...',
          forbidClick: true,
          duration: 0,
          loadingType: 'spinner'
        })

        this.axios({
          method: 'POST',
          url: '/addAddress',
          data
        }).then(result => {
          this.$toast.clear();
          
          this.$toast(result.data.msg);
          if (result.data.code == 9000) {
            setTimeout(() => {
              this.$router.go(-1);
            }, 2000)
          }
        }).catch(err => {
          this.$toast.clear();
          
        })
      },

      //编辑地址  (content：表单内容)
      editAddress(content) {

        //获取token字符串
        let tokenString = localStorage.getItem('CSTK');
        if (!tokenString) {
          return this.$router.push({name: 'Login'});
        }

        
        
        

        //判断地址是否被修改
        let isModify = false;
        // 循环 判断初始值是否和表单的值相同，
        for (let key in this.addressInfo) {
          if (this.addressInfo[key] != content[key]) {
            //已经被修改
            isModify = true;
            break;
          }
        }



        if (isModify) {
          //发起修改地址请求
          

          //复制对象
          let addressData = Object.assign({}, content);
          delete addressData.country;
          delete addressData.id;
          addressData.aid = this.aid;

          addressData.tokenString = tokenString;
          addressData.appkey = this.appkey;
          addressData.isDefault = Number(addressData.isDefault);

          //序列化参数
          let data = utils.queryString(addressData);

          this.$toast.loading({
            message: '加载中...',
            forbidClick: true,
            duration: 0,
            loadingType: 'spinner'
          })

          this.axios({
            method: 'POST',
            url: '/editAddress',
            data
          }).then(result => {
            this.$toast.clear();
            
            // 编辑地址成功后过一秒跳转至地址界面
            this.$toast(result.data.msg);
            if (result.data.code == 30000) {
              setTimeout(() => {
                this.$router.push({name: 'Address'});
              }, 1000)
            }
          }).catch(err => {
            this.$toast.clear();
            
          })

        } else {
          this.$router.push({name: 'Address'});
        }

      },

      //保存地址  (content：表单内容)
      saveAddress(address) {
        if (this.aid) {
          //编辑地址
          this.editAddress(address);
        } else {
          this.newAddress(address);
        }
      },

      //根据aid查询地址数据  (设置编辑地址的初始值)
      getAddressDataByAid() {
        //获取token字符串
        let tokenString = localStorage.getItem('CSTK');
        if (!tokenString) {
          return this.$router.push({name: 'Login'});
        }

        this.$toast.loading({
          message: '加载中...',
          forbidClick: true,
          duration: 0,
          loadingType: 'spinner'
        })

        this.axios({
          method: 'GET',
          url: '/findAddressByAid',
          params: {
            appkey: this.appkey,
            tokenString,
            aid: this.aid
          }
        }).then(result => {
          this.$toast.clear();
          
          if (result.data.code == 40000) {
            for (let key in this.addressInfo) {
              
              if (key == 'id') {
                this.addressInfo[key] = result.data.result[0].aid;
                continue;
              }
              if (key == 'isDefault') {
                this.addressInfo[key] = Boolean(result.data.result[0][key]);
                continue;
              }

              this.addressInfo[key] = result.data.result[0][key];
            }
          }
        }).catch(err => {
          this.$toast.clear();
          
        })
      },

      //删除地址
      removeAddress() {

        //获取token字符串
        let tokenString = localStorage.getItem('CSTK');
        if (!tokenString) {
          return this.$router.push({name: 'Login'});
        }

        //参数序列化
        let data = utils.queryString({
          appkey: this.appkey,
          tokenString,
          aid: this.aid
        });

        this.$toast.loading({
          message: '加载中...',
          forbidClick: true,
          duration: 0,
          loadingType: 'spinner'
        })

        this.axios({
          method: 'POST',
          url: '/deleteAddress',
          data
        }).then(result => {
          this.$toast.clear();
          
          this.$toast(result.data.msg);
          if (result.data.code == 10000) {
            setTimeout(() => {
              this.$router.push({name: 'Address'});
            }, 1000)
          }
        }).catch(err => {
          this.$toast.clear();
          
        })
      }
    }
  }
</script>
