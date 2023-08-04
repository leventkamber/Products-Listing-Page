<template>
  <!-- Header -->
  <section id="header">
    <div id="mobile">
      <i id="bar" class="fas fa-outdent" @click="toggleNavbar"></i>
    </div>
    <div :class="isActive ? 'active' : ''">
      <ul id="navbar">
        <li>
          <a
            href="all"
            @click="
              filterByCategory('#');
              closeNavbar();
            "
            >All</a
          >
        </li>
        <li>
          <a
            href="#"
            @click="
              filterByCategory('Electronics');
              closeNavbar();
            "
            >Electronics</a
          >
        </li>
        <li>
          <a
            href="#"
            @click="
              filterByCategory('Cosmetics');
              closeNavbar();
            "
            >Cosmetics</a
          >
        </li>
        <li>
          <a
            href="#"
            @click="
              filterByCategory('Foods');
              closeNavbar();
            "
            >Foods</a
          >
        </li>
        <li>
          <a
            href="#"
            @click="
              filterByCategory('Other');
              closeNavbar();
            "
            >Other</a
          >
        </li>
        <a href="#" id="close" @click="closeNavbar"
          ><i class="fa fa-times"></i
        ></a>
      </ul>
    </div>

    <a href="#"><img src="@/assets/logo.png" class="logo" alt="" /></a>
  </section>

  <!-- Main content -->
  <div class="main-content">
    <!-- Sorting section     -->
    <div class="top-row">
      <div class="category-name" v-if="displayedCategory !== 'All'">
        <h2>{{ displayedCategory }}</h2>
        <div>{{ productCount }} Products</div>
      </div>
      <div class="sorting-container">
        <label for="sortSelect">Sort By:</label>
        <select id="sortSelect" v-model="sortOption" @change="sortProducts">
          <option value="asc">Price Low to High</option>
          <option value="desc">Price High to Low</option>
          <option value="name-asc">Name A-Z</option>
          <option value="name-desc">Name Z-A</option>
        </select>
      </div>
    </div>

    <div class="middle-row">
      <!-- Filter Section -->
      <div class="filter-section">
        <h3>Filter By Price</h3>
        <select v-model="selectedPrice" @change="filterProducts">
          <option value="all">All</option>
          <option value="0-99">$0 - $99</option>
          <option value="100-499">$100 - $499</option>
          <option value="500-999">$500 - $999</option>
          <option value="1000-2000">$1000 - $2000</option>
        </select>
      </div>
      <!-- Product grid -->
      <div class="product-grid">
        <div v-if="allVisibleProducts.length === 0" class="no-products-message">
          No products found.
        </div>
        <div
          v-for="product in allVisibleProducts"
          :key="product.id"
          class="col-md-4 mb-4"
        >
          <div class="card">
            <img
              :src="product.thumbnail"
              class="card-img-top"
              alt="Product Image"
            />
            <div class="card-body">
              <h4 class="card-title">
                {{
                  product.title[0].toUpperCase() +
                  product.title.substring(1).toLowerCase()
                }}
              </h4>
              <p class="card-text">{{ product.brand }}</p>
              <p class="card-text">{{ product.description }}</p>
              <h5 class="card-text">Price: ${{ product.price }}</h5>
              <p class="card-text">
                Discount: {{ product.discountPercentage }}%
              </p>
              <StarRating :rating="product.rating" class="star" />
              <p class="card-text">Availaible in stock: {{ product.stock }}</p>
            </div>
            <div class="button-container">
              <button class="add-to-cart-btn" @click="addToCart">
                Add to Cart
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="load-more-container" v-if="hasMoreProducts">
      <button @click="loadMore" class="load-more-btn">Load More</button>
    </div>
  </div>

  <!-- Footer -->
  <footer class="section-p1">
    <div class="col">
      <img class="footer-logo" src="@/assets/logo.png" alt="" />
      <h4>Contact</h4>
      <p><strong>Address: </strong> 123 Simple Road, Street 12, City</p>
      <p><strong>Phone: </strong> +41 41 369 79 79</p>
      <p><strong>Hours: </strong> Mon - Fri 09:00 - 18:00</p>
      <div class="follow">
        <h4>Follow us</h4>
        <div class="icon">
          <i class="fab fa-facebook-f"></i>
          <i class="fab fa-twitter"></i>
          <i class="fab fa-instagram"></i>
          <i class="fab fa-pinterest-p"></i>
          <i class="fab fa-youtube"></i>
          <i class="fab fa-linkedin"></i>
        </div>
      </div>
    </div>
    <div class="col">
      <h4>About</h4>
      <a href="#">About us</a>
      <a href="#">Delivery</a>
      <a href="#">Privacy Policy</a>
      <a href="#">Terms & Conditions</a>
      <a href="#">Contact Us</a>
    </div>
    <div class="col">
      <h4>My Account</h4>
      <a href="#">Sign in</a>
      <a href="#">View Cart</a>
      <a href="#">My Wishlist</a>
      <a href="#">Track My Order</a>
      <a href="#">Help</a>
    </div>
    <div class="install">
      <h4>Install App</h4>
      <p>From App Store or Google Play</p>
      <div class="row">
        <img src="@/assets/app-store.svg" alt="" />
        <img src="@/assets/google-play.svg" alt="" />
      </div>
      <p>Secured Payment Gateways</p>
      <img class="secure-pay" src="@/assets/secure-pay.png" alt="" />
    </div>
    <div class="copyright">
      <p>© 2023 Product Listing Page</p>
    </div>
  </footer>
</template>

<script>
import StarRating from "./StarRating.vue";
import Swal from "sweetalert2";

export default {
  components: { StarRating },
  data() {
    return {
      products: [],
      sortOption: "asc", // sorted ascending by default
      selectedPrice: "all",
      productsPerPage: 12, // Number of products to display per page
      currentPage: 1,
      isActive: false,
      selectedCategory: "All",
      displayedCategory: "",
      showSuccessAlert: false,
    };
  },
  created() {
    // Fetch data from the API when the component is created
    this.fetchProductData();
  },

  computed: {
    // Computed property for displaying the product count
    productCount() {
      return this.allVisibleProducts.length;
    },
    sortedProducts() {
      if (this.sortOption === "asc") {
        return this.products.slice().sort((a, b) => a.price - b.price);
      } else if (this.sortOption === "desc") {
        return this.products.slice().sort((a, b) => b.price - a.price);
      } else if (this.sortOption === "name-asc") {
        return this.products
          .slice()
          .sort((a, b) => a.title.localeCompare(b.title));
      } else if (this.sortOption === "name-desc") {
        return this.products
          .slice()
          .sort((a, b) => b.title.localeCompare(a.title));
      } else {
        return this.products.slice(); // Default: No sorting, return original order
      }
    },
    allVisibleProducts() {
      const startIndex = 0; // Start from the first product
      const endIndex = this.currentPage * this.productsPerPage;

      let sortedProducts = this.sortedProducts;
      //filter by price range
      if (this.selectedPrice !== "all") {
        const [minPrice, maxPrice] = this.selectedPrice.split("-").map(Number);
        sortedProducts = sortedProducts.filter(
          (product) => product.price >= minPrice && product.price <= maxPrice
        );
      }

      //filter by category
      if (this.selectedCategory !== "All") {
        if (this.selectedCategory === "Electronics") {
          sortedProducts = sortedProducts.filter(
            (product) =>
              product.category === "smartphones" ||
              product.category === "laptops"
          );
        } else if (this.selectedCategory === "Cosmetics") {
          sortedProducts = sortedProducts.filter(
            (product) =>
              product.category === "fragrances" ||
              product.category === "skincare"
          );
        } else if (this.selectedCategory === "Foods") {
          sortedProducts = sortedProducts.filter(
            (product) => product.category === "groceries"
          );
        } else {
          sortedProducts = sortedProducts.filter(
            (product) =>
              ![
                "smartphones",
                "laptops",
                "fragrances",
                "skincare",
                "groceries",
              ].includes(product.category)
          );
        }
      }

      return sortedProducts.slice(startIndex, endIndex);
    },

    hasMoreProducts() {
      const totalVisibleProducts = this.currentPage * this.productsPerPage;
      return (
        totalVisibleProducts < this.sortedProducts.length &&
        this.allVisibleProducts.length >= this.productsPerPage
      );
    },
  },
  methods: {
    async fetchProductData() {
      try {
        const response = await fetch("https://dummyjson.com/products");
        if (!response.ok) {
          throw new Error("Network response was not ok");
        }
        const data = await response.json();
        this.products = data.products; // Get the products array from the API response
        console.log(this.products.slice()[0]["title"]);
      } catch (error) {
        console.error("Error fetching product data:", error);
      }
    },
    loadMore() {
      this.currentPage += 1;
    },
    toggleNavbar() {
      this.isActive = !this.isActive;
    },
    closeNavbar() {
      this.isActive = false;
    },
    filterByCategory(category) {
      this.selectedCategory = category;
      this.displayedCategory = category;
      this.closeNavbar();
    },
    filterProducts() {
      this.currentPage = 1; // Reset the current page when filtering
    },

    addToCart() {
      Swal.fire({
        position: "center",
        icon: "success",
        title: "Product added to cart!",
        showConfirmButton: false,
        timer: 1500,
      });
    },
  },
};
</script>

<style></style>
