<template>
  <div id="app">

  <div class="container mt-5">
      <h1 class="text-primary mb-4">products List</h1>

      <div class="row add-product" v-if="!isEditing">
          <div class="col-xl-3 col-md-6 mb-3 mb-xl-0">
            <input class="form-control" type="text" v-model="currentProduct.title" placeholder="product title...">
          </div>
          <div class="col-xl-3 col-md-6 mb-3 mb-xl-0">
            <input class="form-control" type="text" v-model="currentProduct.category" placeholder="product category...">
          </div>
          <div class="col-xl-4 col-md-6 mb-3 mb-xl-0">
            <input class="form-control" type="file" @change="onFileChange" ref="fileupload">
          </div>
          <div class="col-xl-2 col-md-6 mb-3 mb-xl-0 text-start">
            <input class="btn px-3 btn-success" type="submit" value="ADD NEW PRODUCT" @click="createProduct">
          </div>
        </div>

        
        <div class="row add-product" v-else>
          <div class="col-xl-3 col-md-6 mb-3 mb-xl-0">
            <input class="form-control" type="text" v-model="currentProduct.title">
          </div>
          <div class="col-xl-3 col-md-6 mb-3 mb-xl-0">
            <input class="form-control" type="text" v-model="currentProduct.category">
          </div>
          <div class="col-xl-4 col-md-6 mb-3 mb-xl-0">
            <input class="form-control" type="file" @change="onFileChange" ref="fileupload">
          </div>
          <div class="col-xl-2 col-md-6 mb-3 mb-xl-0 text-start">
            <input class="btn px-3 btn-success" type="submit" value="UPDATE PRODUCT" @click="updateProduct">
          </div>
        </div>

          <div class="d-flex justify-content-center align-items-center py-3">
              <div class="grid-box">
                  <div>
                      <button class="btn btn-secondary ms-3 float-start" @click="gridShow('list')"><i class="fas fa-list"></i></button>
                      <button class="btn btn-primary ms-3 float-end" @click="gridShow('grid')"><i class="fas fa-th"></i></button>
                  </div>
              </div>
          </div>

          <div v-if="this.loading" class="h-100 d-flex justify-content-center align-items-center">
              <Spinner/>
          </div>

        <div class="row">

          <template v-if="this.grid" >
            <div class="col-md-4" v-for="(product, index) in products" v-bind:key="product" >
                <div class="card mb-4">

                    <div class="card-image-div">
                        <img class="card-image" :src="product.image || defaultImage" alt="product"/>
                    </div>

                    <div class="card-body d-flex flex-column justify-content-center">
                        <div class="text-center">
                            <p class="card-text mb-1">{{product.title}}</p>
                            <small class="text-muted">( {{product.category}} )</small>
                        </div>
                        <div class="d-flex justify-content-between align-items-center mt-5">
                            <div class="card-edit" @click="editProduct(index, product)">
                                <i class="icon far fa-edit fa-2x text-primary"></i>
                            </div>
                            <div class="card-delete" @click="deleteProduct(index)">
                              <i class="icon fas fa-trash fa-2x text-danger mt-2"></i>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
          </template>

          <template v-if="!this.grid" >
            <div class="col-lg-6" v-for="(product, index) in products" v-bind:key="product" >
                <div class="card-list mb-4 d-flex flex-row justify-content-around align-items-center">
                    <div class="w-20 card-list-image-div">
                        <img class="card-list-image" :src="product.image || defaultImage" alt="product"/>
                    </div>

                    <div class="w-80 card-list-text text-center">
                        <p class="mb-1">{{product.title}}</p>
                        <small class="text-muted">( {{product.category}} )</small>
                    </div>

                    <div class="w-20 d-flex flex-column justify-content-around align-items-center h-100">
                        <div class="card-edit"  @click="editProduct(index, product)">
                          <i class="icon far fa-edit fa-2x text-primary"></i>
                        </div>
                        <div class="card-delete"  @click="deleteProduct(index)">
                          <i class="icon fas fa-trash fa-2x text-danger mt-2"></i>
                        </div>
                    </div>

                </div>
            </div>
          </template>

    </div>
  </div>

  </div>
</template>

<script>
import './Products.scss';
import Spinner from '../Spinner/Spinner.vue';
import defaultProductImage from '../../assets/images/default-product.jpg';
import axios from 'axios';

const baseURL = "https://fakestoreapi.com/products";

export default {
  name: 'products',
  components: {
    Spinner
  },
  data() {
    return {
      products: [],
      loading:true,
      grid:true,
      isEditing: false,
      currentProduct: {},
      selectedproduct: null,
      defaultImage:defaultProductImage,
    }
  },
  async created() {
    try {
      const res = await axios.get(baseURL)
      this.products = res.data;
      this.loading = false;
    } catch(e) {
      console.error(e);
    }
  },
  methods: {
    // show grid/list products
    gridShow (gridShowType) {
      if (gridShowType === "grid")
      this.grid = true;

      else if (gridShowType === "list")
      this.grid = false;
    },

    // upload iamge
    onFileChange(e) {
      let files = e.target.files || e.dataTransfer.files;
      if (!files.length)
        return;
      this.createImage(files[0]);
    },

    createImage(file) {
      let reader = new FileReader();
      reader.onload = (e) => {
        this.currentProduct.image = e.target.result;
      };
      reader.readAsDataURL(file);
    },

    // CRUD operations
    createProduct() {
      if (!this.currentProduct.title || !this.currentProduct.category )
      alert('please enter product title and category');

      else{
        this.products.unshift(this.currentProduct);
        this.currentProduct = {};
        this.$refs.fileupload.value = null;
      }
    },

    deleteProduct(index) {
        this.products.splice(index, 1)
    },

    updateProduct() {
      if (!this.currentProduct.title || !this.currentProduct.category )
      alert('please enter product title and category');

      else{
        this.products.splice(this.selectedproduct, 1, this.currentProduct)
        this.currentProduct = {};
        this.isEditing = false
      }
    },

    editProduct(index, product) {
        this.isEditing = true;
        this.currentProduct = product;
        this.selectedproduct = index;
    }
  }
}
</script>