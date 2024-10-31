<template>
  <v-card>
      <v-card-title>
        <v-btn color="primary" @click="show=true">新增品牌</v-btn>
        <v-spacer></v-spacer>
        <v-text-field
          append-icon="search"
          label="搜索"
          single-line
          hide-details
          v-model="search"
        />
      </v-card-title>
      <v-divider></v-divider>
        <v-data-table
          :headers="headers"
          :items="brands"
          :loading="loading"
          :pagination.sync="pagination"
          :total-items="totalBrands"
          class="elevation-1"
        >
          <template v-slot:items="props">
            <td>{{ props.item.id }}</td>
            <td class="text-xs-center">{{ props.item.name }}</td>
            <td class="text-xs-center"><img width="100" v-if="props.item.image"  :src="props.item.image"/><span v-else>无</span></td>
            <td class="text-xs-center">{{ props.item.letter }}</td>
            <td class="justify-center layout px-0">
              <v-btn icon @click="editBrand(props.item)"><i class="el-icon-edit"/></v-btn>
              <v-btn icon><i class="el-icon-delete"/></v-btn>
            </td>
          </template>
          <template slot="no-data">
            <v-alert :value="true" color="red" icon="warning">
              对不起，没有查询到任何数据 :(
            </v-alert>
          </template>
          <template slot="pageText" slot-scope="props">
            共{{props.itemsLength}}条,当前:{{ props.pageStart }} - {{ props.pageStop }}
          </template>
        </v-data-table>

    <!--弹出的对话框-->
    <v-dialog max-width="500" v-model="show" persistent>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>新增品牌</v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口的按钮-->
          <v-btn icon @click="show=false"><v-icon>close</v-icon></v-btn>

        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5">
          <he-brand-form @close="closeWindow" :oldBrand="oldBrand"></he-brand-form>
        </v-card-text>
      </v-card>
    </v-dialog>

  </v-card>

</template>

<script>
  // 导入自定义的表单组件
  import HeBrandForm from './HeBrandForm'

  export default {
    name: "HeBrand",
    components:{
      HeBrandForm
    },
    data() {
      return {
        totalBrands:0,//品牌个数
        search: '',// 过滤字段
        pagination: {},
        selected: [],
        headers: [
          {
            text: 'Id',
            align: 'center',
            sortable: true,
            value: 'id'
          },
          {text: '名称', value: 'name', sortable: false, align: 'center',},
          {text: 'LOGO', value: 'logo', sortable: false, align: 'center',},
          {text: '首字母', value: 'letter', sortable: true, align: 'center',},
          {text: '操作', value: '-', sortable: false, align: 'center',}
        ],
        brands: [],
        loading: true,
        show:false,
        oldBrand:{}
      }
    },
    created(){
      this.getDataFromServer();

    },
    watch:{
      pagination:{
        deep:true,
        handler(){
          this.getDataFromServer();
        }
      },
      search(){
        this.getDataFromServer();
      }
    },

    methods: {
      editBrand(brand){

        // 查询商品分类信息，进行回显
        this.$http.get("/item/category/bid/" + brand.id).then(resp => {

          this.oldBrand = brand;
          this.oldBrand.categories = resp.data;

          this.isEdit = true;
          this.show=true;
        })

      },
      getDataFromServer() { // 从服务的加载数据的方法。
        this.$http.get("/item/brand/page",{
          params:{
            "page":this.pagination.page,
            "rows":this.pagination.rowsPerPage,
            "key":this.search,
            "sortBy":this.pagination.sortBy,
            "desc":this.pagination.descending
          }
        }).then((resp)=>{
            this.brands = resp.data.items;//当前页的数据
            this.totalBrands = resp.data.total;//总记录数
        })

        this.loading = false;
      },
      closeWindow(){
        // 关闭窗口
        this.show = false;
        // 重新加载数据
        this.getDataFromServer();
      }

    }
  }
</script>

<style scoped>

</style>
