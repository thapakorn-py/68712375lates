<template>
  <div class="shop-page">
    <main class="shop-content">
      <h2 class="page-title">รายการสินค้า</h2>

      <div v-if="loading" class="loading-box">กำลังโหลดข้อมูล...</div>
      <div v-else-if="error" class="alert alert-danger">{{ error }}</div>

      <div v-else class="product-grid">
        <article v-for="item in assets" :key="item.asset_id" class="product-card">
          <div class="product-image-wrap">
            <img
              v-if="item.image"
              :src="imageUrl(item.image)"
              :alt="item.asset_name"
              class="product-image"
              @error="hideBrokenImage"
            />
            <div v-else class="product-image placeholder">No Image</div>
          </div>

          <div class="product-name">{{ item.asset_name }}</div>
          <div class="product-price">{{ formatPrice(item.price) }} บาท</div>

          <button class="buy-btn" type="button">สั่งซื้อ</button>
        </article>
      </div>
    </main>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import { API_BASE } from "@/config";

export default {
  name: "AssetCatalog",
  setup() {
    const assets = ref([]);
    const loading = ref(true);
    const error = ref("");
    const apiBase = API_BASE.replace(/\/$/, "");

    const imageUrl = (image) => {
      if (!image) return "";
      return `${apiBase}/uploads/${encodeURIComponent(image)}`;
    };

    const formatPrice = (value) => {
      const number = Number(value);
      if (Number.isNaN(number)) return "0";
      return number.toLocaleString("en-US", { minimumFractionDigits: 2, maximumFractionDigits: 2 });
    };

    const hideBrokenImage = (event) => {
      event.target.style.display = "none";
    };

    const fetchAssets = async () => {
      try {
        const response = await fetch(`${apiBase}/api_assets.php`);

        if (!response.ok) {
          throw new Error("ไม่สามารถดึงข้อมูลได้");
        }

        const result = await response.json();
        if (result.success && Array.isArray(result.data)) {
          assets.value = result.data;
        } else {
          throw new Error(result.message || "ไม่สามารถดึงข้อมูลสินค้าได้");
        }
      } catch (err) {
        error.value = err instanceof Error ? err.message : "ไม่สามารถดึงข้อมูลสินค้าได้";
      } finally {
        loading.value = false;
      }
    };

    onMounted(fetchAssets);

    return {
      assets,
      loading,
      error,
      imageUrl,
      formatPrice,
      hideBrokenImage
    };
  }
};
</script>

<style scoped>
.shop-page {
  min-height: 100vh;
  background: #f3f3f3;
  font-family: "Segoe UI", Tahoma, sans-serif;
}

.shop-content {
  max-width: 1180px;
  margin: 0 auto;
  padding: 24px 16px 40px;
}

.page-title {
  text-align: center;
  font-size: 2.2rem;
  font-weight: 600;
  margin: 18px 0 26px;
  color: #111;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(220px, 1fr));
  gap: 26px 18px;
  justify-items: center;
}

.product-card {
  width: 100%;
  max-width: 255px;
  min-height: 380px;
  background: #f7f7f7;
  border: 1px solid #cfcfcf;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 18px 16px 14px;
  box-sizing: border-box;
}

.product-image-wrap {
  width: 100%;
  height: 180px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f1f1f1;
  border-radius: 8px;
  overflow: hidden;
}

.product-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.product-image.placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
  font-size: 0.9rem;
}

.product-name {
  text-align: center;
  font-size: 1.2rem;
  font-weight: 400;
  color: #111;
  margin-top: 10px;
  min-height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.product-price {
  text-align: center;
  font-size: 1.1rem;
  color: #171717;
  margin-top: 4px;
}

.buy-btn {
  width: 100%;
  border: 0;
  border-radius: 8px;
  background: #0d83ff;
  color: #fff;
  font-weight: 600;
  padding: 10px 0;
  cursor: pointer;
  margin-top: 12px;
}

.loading-box {
  padding: 28px;
  text-align: center;
  color: #555;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 10px;
}

@media (max-width: 980px) {
  .product-grid {
    grid-template-columns: repeat(2, minmax(220px, 1fr));
  }
}

@media (max-width: 640px) {
  .shop-header {
    flex-direction: column;
    height: auto;
    padding: 12px 16px;
    gap: 8px;
  }

  .main-nav {
    justify-content: center;
  }

  .product-grid {
    grid-template-columns: 1fr;
  }
}
</style>
