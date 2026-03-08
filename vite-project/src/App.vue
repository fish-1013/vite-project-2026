<template>
  <div class="cart-container">
    <h1>🛒 我的淘宝购物车</h1>

    <!-- 购物车为空时的提示 -->
    <div v-if="cartItems.length === 0" class="empty-cart">
      <p>购物车空空如也，快去选购吧！</p>
    </div>

    <!-- 商品列表 -->
    <div v-else class="cart-list">
      <!-- 表头 -->
      <div class="cart-header">
        <span class="col-check">选择</span>
        <span class="col-info">商品信息</span>
        <span class="col-price">单价</span>
        <span class="col-quantity">数量</span>
        <span class="col-total">小计</span>
        <span class="col-action">操作</span>
      </div>

      <!-- 商品项循环 -->
      <div v-for="item in cartItems" :key="item.id" class="cart-item">
        <!-- 复选框 -->
        <input 
          type="checkbox" 
          v-model="item.selected" 
          @change="calculateTotal"
        />

        <!-- 商品信息 -->
        <div class="item-info">
          <img :src="item.image" alt="商品图" class="item-image" />
          <div class="item-name">{{ item.name }}</div>
          <div class="item-spec">{{ item.spec }}</div>
        </div>

        <!-- 单价 -->
        <div class="item-price">¥{{ item.price.toFixed(2) }}</div>

        <!-- 数量控制器 -->
        <div class="item-quantity">
          <button @click="decreaseQuantity(item)" :disabled="item.count <= 1">-</button>
          <span>{{ item.count }}</span>
          <button @click="increaseQuantity(item)">+</button>
        </div>

        <!-- 小计 -->
        <div class="item-total">
          ¥{{ (item.price * item.count).toFixed(2) }}
        </div>

        <!-- 删除按钮 -->
        <button class="btn-delete" @click="removeItem(item.id)">删除</button>
      </div>
    </div>

    <!-- 底部结算栏 -->
    <div v-if="cartItems.length > 0" class="cart-footer">
      <div class="footer-left">
        <label>
          <input type="checkbox" v-model="selectAll" @change="toggleSelectAll" /> 全选
        </label>
        <button class="btn-delete-batch" @click="deleteSelected">删除选中</button>
      </div>
      <div class="footer-right">
        <span class="total-text">已选 <strong>{{ selectedCount }}</strong> 件</span>
        <span class="total-price">合计：<strong class="price-highlight">¥{{ totalPrice.toFixed(2) }}</strong></span>
        <button class="btn-checkout" @click="checkout">结算</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue';

// 1. 模拟数据 (实际项目中通常从 API 获取)
const cartItems = reactive([
  {
    id: 1,
    name: "Vue 3 实战教程书籍",
    spec: "精装版 / 黑色封面",
    price: 59.00,
    count: 1,
    selected: true,
    image: "https://via.placeholder.com/80?text=Book"
  },
  {
    id: 2,
    name: "机械键盘 Keychron K2",
    spec: "红轴 / RGB背光",
    price: 499.00,
    count: 1,
    selected: false,
    image: "https://via.placeholder.com/80?text=Keyboard"
  },
  {
    id: 3,
    name: "无线鼠标 Logitech MX",
    spec: "灰色 / 静音点击",
    price: 299.00,
    count: 2,
    selected: true,
    image: "https://via.placeholder.com/80?text=Mouse"
  }
]);

// 2. 增加数量
const increaseQuantity = (item) => {
  item.count++;
  calculateTotal();
};

// 3. 减少数量
const decreaseQuantity = (item) => {
  if (item.count > 1) {
    item.count--;
    calculateTotal();
  }
};

// 4. 删除单个商品
const removeItem = (id) => {
  if(confirm("确定要删除这个商品吗？")) {
    const index = cartItems.findIndex(item => item.id === id);
    if (index !== -1) {
      cartItems.splice(index, 1);
      calculateTotal();
    }
  }
};

// 5. 删除选中商品
const deleteSelected = () => {
  if (selectedCount.value === 0) {
    alert("请先选择要删除的商品");
    return;
  }
  if(confirm(`确定要删除选中的 ${selectedCount.value} 件商品吗？`)) {
    for (let i = cartItems.length - 1; i >= 0; i--) {
      if (cartItems[i].selected) {
        cartItems.splice(i, 1);
      }
    }
    calculateTotal();
  }
};

// 6. 全选/反选逻辑
const selectAll = computed({
  get: () => cartItems.length > 0 && cartItems.every(item => item.selected),
  set: (val) => cartItems.forEach(item => item.selected = val)
});

const toggleSelectAll = () => {
  const newVal = !selectAll.value;
  cartItems.forEach(item => item.selected = newVal);
  calculateTotal();
};

// 7. 计算选中商品总数
const selectedCount = computed(() => {
  return cartItems.filter(item => item.selected).reduce((sum, item) => sum + item.count, 0);
});

// 8. 计算总价 (核心逻辑)
const totalPrice = ref(0);

const calculateTotal = () => {
  totalPrice.value = cartItems
    .filter(item => item.selected)
    .reduce((sum, item) => sum + item.price * item.count, 0);
};

// 初始化计算一次
calculateTotal();

// 9. 结算按钮
const checkout = () => {
  if (totalPrice.value === 0) {
    alert("请选择至少一件商品进行结算");
    return;
  }
  alert(`结算成功！共支付 ¥${totalPrice.value.toFixed(2)}`);
};
</script>

<style scoped>
/* 简单的美化样式 */
.cart-container {
  max-width: 900px;
  margin: 20px auto;
  font-family: Arial, sans-serif;
  color: #333;
}

h1 {
  text-align: center;
  color: #ff5000; /* 淘宝橙 */
}

.empty-cart {
  text-align: center;
  padding: 50px;
  background: #f9f9f9;
  border-radius: 8px;
}

.cart-header, .cart-item {
  display: grid;
  grid-template-columns: 50px 1fr 100px 120px 100px 80px;
  align-items: center;
  padding: 15px 10px;
  border-bottom: 1px solid #eee;
  gap: 10px;
}

.cart-header {
  background: #f5f5f5;
  font-weight: bold;
  border-radius: 8px 8px 0 0;
}

.item-info {
  display: flex;
  align-items: center;
  gap: 15px;
}

.item-image {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 4px;
  border: 1px solid #eee;
}

.item-name {
  font-weight: bold;
  margin-bottom: 5px;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.item-spec {
  font-size: 12px;
  color: #999;
  background: #f9f9f9;
  padding: 2px 6px;
  border-radius: 4px;
  display: inline-block;
}

.item-quantity button {
  width: 28px;
  height: 28px;
  border: 1px solid #ddd;
  background: #fff;
  cursor: pointer;
  font-size: 16px;
}

.item-quantity button:disabled {
  color: #ccc;
  cursor: not-allowed;
}

.item-quantity span {
  display: inline-block;
  width: 40px;
  text-align: center;
}

.btn-delete, .btn-delete-batch {
  background: none;
  border: none;
  color: #999;
  cursor: pointer;
  text-decoration: underline;
}

.btn-delete:hover, .btn-delete-batch:hover {
  color: #ff5000;
}

.cart-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background: #fff;
  border-top: 2px solid #ff5000;
  margin-top: 20px;
  position: sticky;
  bottom: 0;
}

.footer-left, .footer-right {
  display: flex;
  align-items: center;
  gap: 20px;
}

.total-price {
  font-size: 18px;
}

.price-highlight {
  color: #ff5000;
  font-size: 24px;
  font-weight: bold;
}

.btn-checkout {
  background: linear-gradient(to right, #ff9000, #ff5000);
  color: white;
  border: none;
  padding: 12px 40px;
  font-size: 18px;
  font-weight: bold;
  border-radius: 20px;
  cursor: pointer;
  transition: transform 0.1s;
}

.btn-checkout:active {
  transform: scale(0.98);
}

/* 移动端适配简化 */
@media (max-width: 600px) {
  .cart-header, .cart-item {
    grid-template-columns: 40px 1fr 80px;
    grid-template-rows: auto auto;
    gap: 10px;
  }
  .col-price, .col-quantity, .col-total, .col-action {
    display: none; /* 简化显示，实际项目需重新设计布局 */
  }
  .item-quantity, .item-total, .btn-delete {
    grid-column: 2 / 4;
    justify-self: start;
  }
}
</style>