<template>
  <div class="api-interface">
    <!-- Role Selection -->
    <div class="role-selector">
      <button @click="setRole('admin')" :class="{ active: currentRole === 'admin' }">Admin</button>
      <button @click="setRole('supplier')" :class="{ active: currentRole === 'supplier' }">Supplier</button>
      <button @click="setRole('customer')" :class="{ active: currentRole === 'customer' }">Customer</button>
    </div>

    <!-- Auth Section -->
    <div class="auth-section">
      <div class="auth-tabs">
        <button @click="authTab = 'login'" :class="{ active: authTab === 'login' }">Login</button>
        <button @click="authTab = 'register'" :class="{ active: authTab === 'register' }">Register</button>
      </div>

      <!-- Login Form -->
      <div v-if="authTab === 'login'" class="auth-form">
        <div class="form-group">
          <label>Email:</label>
          <input v-model="loginEmail" type="email" placeholder="user@example.com">
        </div>
        <div class="form-group">
          <label>Password:</label>
          <input v-model="loginPassword" type="password" placeholder="Your password">
        </div>
        <button @click="login" class="auth-button">Login</button>
        <div class="response">{{ authResponse }}</div>
      </div>

      <!-- Registration Form -->
      <div v-if="authTab === 'register'" class="auth-form">
        <div class="form-row">
          <div class="form-group">
            <label>Name:</label>
            <input v-model="registerName" type="text">
          </div>
          <div class="form-group">
            <label>Email:</label>
            <input v-model="registerEmail" type="email">
          </div>
        </div>
        <div class="form-row">
          <div class="form-group">
            <label>Password:</label>
            <input v-model="registerPassword" type="password">
          </div>
          <div class="form-group">
            <label>Confirm Password:</label>
            <input v-model="registerPassword2" type="password">
          </div>
        </div>
        <div class="form-group">
          <label>Role:</label>
          <select v-model="registerRole">
            <option value="customer">Customer</option>
            <option value="supplier">Supplier</option>
          </select>
        </div>
        <div v-if="registerRole === 'supplier'" class="form-group">
          <label>Categories (comma separated):</label>
          <input v-model="registerCategories" type="text">
        </div>
        <button @click="register" class="auth-button">Register</button>
        <div class="response">{{ registerResponse }}</div>
      </div>
    </div>

    <!-- Admin Interface -->
    <div v-if="currentRole === 'admin'" class="role-interface">
      <div class="tab-buttons">
        <button @click="adminTab = 'users'" :class="{ active: adminTab === 'users' }">Users</button>
        <button @click="adminTab = 'orders'" :class="{ active: adminTab === 'orders' }">Orders</button>
        <button @click="adminTab = 'suppliers'" :class="{ active: adminTab === 'suppliers' }">Suppliers</button>
        <button @click="adminTab = 'recommendations'" :class="{ active: adminTab === 'recommendations' }">Recommendations</button>
      </div>

      <!-- Users Management -->
      <div v-if="adminTab === 'users'" class="tab-content">
        <div class="endpoint">
          <h3>Get All Users</h3>
          <div class="form-group">
            <label>Query Parameters:</label>
            <input v-model="adminUsersQuery" placeholder="?role=customer">
          </div>
          <button @click="adminGetUsers" class="action-button">Get Users</button>
          <div class="response">{{ adminUsersResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Update User</h3>
          <div class="form-group">
            <label>User ID:</label>
            <input v-model="adminUpdateUserId" type="number">
          </div>
          <div class="form-group">
            <label>Update Data (JSON):</label>
            <textarea v-model="adminUpdateUserData" placeholder='{"is_active": true, "role": "admin"}'></textarea>
          </div>
          <button @click="adminUpdateUser" class="action-button">Update User</button>
          <div class="response">{{ adminUpdateUserResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Delete User</h3>
          <div class="form-group">
            <label>User ID:</label>
            <input v-model="adminDeleteUserId" type="number">
          </div>
          <button @click="adminDeleteUser" class="action-button delete">Delete User</button>
          <div class="response">{{ adminDeleteUserResponse }}</div>
        </div>
      </div>

      <!-- Orders Management -->
      <div v-if="adminTab === 'orders'" class="tab-content">
        <div class="endpoint">
          <h3>Get All Orders</h3>
          <div class="form-group">
            <label>Query Parameters:</label>
            <input v-model="adminOrdersQuery" placeholder="?status=pending">
          </div>
          <button @click="adminGetOrders" class="action-button">Get Orders</button>
          <div class="response">{{ adminOrdersResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Create Order</h3>
          <div class="form-group">
            <label>Order Data (JSON):</label>
            <textarea v-model="adminCreateOrderData" placeholder='{"product": "Laptop", "quantity": 2, "customer_id": 1}'></textarea>
          </div>
          <button @click="adminCreateOrder" class="action-button">Create Order</button>
          <div class="response">{{ adminCreateOrderResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Update Order</h3>
          <div class="form-group">
            <label>Order ID:</label>
            <input v-model="adminUpdateOrderId" type="number">
          </div>
          <div class="form-group">
            <label>Update Data (JSON):</label>
            <textarea v-model="adminUpdateOrderData" placeholder='{"status": "completed"}'></textarea>
          </div>
          <button @click="adminUpdateOrder" class="action-button">Update Order</button>
          <div class="response">{{ adminUpdateOrderResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Delete Order</h3>
          <div class="form-group">
            <label>Order ID:</label>
            <input v-model="adminDeleteOrderId" type="number">
          </div>
          <button @click="adminDeleteOrder" class="action-button delete">Delete Order</button>
          <div class="response">{{ adminDeleteOrderResponse }}</div>
        </div>
      </div>

      <!-- Suppliers Management -->
      <div v-if="adminTab === 'suppliers'" class="tab-content">
        <div class="endpoint">
          <h3>Get All Suppliers</h3>
          <div class="form-group">
            <label>Query Parameters:</label>
            <input v-model="adminSuppliersQuery" placeholder="?is_verified=true">
          </div>
          <button @click="adminGetSuppliers" class="action-button">Get Suppliers</button>
          <div class="response">{{ adminSuppliersResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Create Supplier</h3>
          <div class="form-group">
            <label>Supplier Data (JSON):</label>
            <textarea v-model="adminCreateSupplierData" placeholder='{"name": "New Supplier", "email": "supplier@example.com"}'></textarea>
          </div>
          <button @click="adminCreateSupplier" class="action-button">Create Supplier</button>
          <div class="response">{{ adminCreateSupplierResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Update Supplier</h3>
          <div class="form-group">
            <label>Supplier ID:</label>
            <input v-model="adminUpdateSupplierId" type="number">
          </div>
          <div class="form-group">
            <label>Update Data (JSON):</label>
            <textarea v-model="adminUpdateSupplierData" placeholder='{"is_verified": true}'></textarea>
          </div>
          <button @click="adminUpdateSupplier" class="action-button">Update Supplier</button>
          <div class="response">{{ adminUpdateSupplierResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Delete Supplier</h3>
          <div class="form-group">
            <label>Supplier ID:</label>
            <input v-model="adminDeleteSupplierId" type="number">
          </div>
          <button @click="adminDeleteSupplier" class="action-button delete">Delete Supplier</button>
          <div class="response">{{ adminDeleteSupplierResponse }}</div>
        </div>
      </div>

      <!-- Recommendations -->
      <div v-if="adminTab === 'recommendations'" class="tab-content">
        <div class="endpoint">
          <h3>Get Recommendations</h3>
          <div class="form-group">
            <label>Order ID:</label>
            <input v-model="adminRecommendationOrderId" type="number">
          </div>
          <div class="form-group">
            <label>Algorithm:</label>
            <select v-model="adminRecommendationAlgorithm">
              <option value="weighted">Weighted</option>
              <option value="fast">Fast</option>
            </select>
          </div>
          <button @click="adminGetRecommendations" class="action-button">Get Recommendations</button>
          <div class="response">{{ adminRecommendationsResponse }}</div>
        </div>
      </div>
    </div>

    <!-- Supplier Interface -->
    <div v-if="currentRole === 'supplier' && authToken" class="role-interface">
      <div class="tab-buttons">
        <button @click="supplierTab = 'profile'" :class="{ active: supplierTab === 'profile' }">Profile</button>
        <button @click="supplierTab = 'orders'" :class="{ active: supplierTab === 'orders' }">Orders</button>
      </div>

      <!-- Profile Management -->
      <div v-if="supplierTab === 'profile'" class="tab-content">
        <!-- Register Supplier (POST /suppliers/) -->
        <div class="endpoint">
          <h3>Register Supplier</h3>
          <div class="form-group">
            <label>Name:</label>
            <input v-model="supplierRegisterName" placeholder="Your company name">
          </div>
          <div class="form-group">
            <label>Email:</label>
            <input v-model="supplierRegisterEmail" type="email" placeholder="supplier@example.com">
          </div>
          <div class="form-group">
            <label>Password:</label>
            <input v-model="supplierRegisterPassword" type="password" placeholder="At least 8 characters">
          </div>
          <div class="form-group">
            <label>Categories (comma-separated):</label>
            <input v-model="supplierRegisterCategories" placeholder="Electronics, Hardware">
          </div>
          <button @click="supplierRegister" class="action-button">Register</button>
          <div class="response">{{ supplierRegisterResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Update Supplier Profile</h3>
          <div class="form-group">
            <label>Name:</label>
            <input v-model="supplierUpdateName" placeholder="New company name">
          </div>
          <div class="form-group">
            <label>Email:</label>
            <input v-model="supplierUpdateEmail" type="email" placeholder="new@example.com">
          </div>
          <div class="form-group">
            <label>Categories (comma-separated):</label>
            <input v-model="supplierUpdateCategories" placeholder="Electronics, Furniture">
          </div>
          <button @click="supplierUpdateProfile" class="action-button">Update Profile</button>
          <div class="response">{{ supplierUpdateResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Delete Supplier Account</h3>
          <div class="form-group">
            <label>Confirm by entering your email:</label>
            <input v-model="supplierDeleteEmail" type="email" placeholder="supplier@example.com">
          </div>
          <button @click="supplierDeleteAccount" class="action-button delete">Delete Account</button>
          <div class="response">{{ supplierDeleteResponse }}</div>
        </div>
      </div>

      <!-- Orders View -->
      <div v-if="supplierTab === 'orders'" class="tab-content">
        <div class="endpoint">
          <h3>View Assigned Orders</h3>
          <div class="form-group">
            <label>Status:</label>
            <select v-model="supplierOrderStatus">
              <option value="">All</option>
              <option value="assigned">Assigned</option>
              <option value="completed">Completed</option>
            </select>
          </div>
          <button @click="supplierGetOrders" class="action-button">Get Orders</button>
          <div class="response">{{ supplierOrdersResponse }}</div>
        </div>
      </div>
    </div>

    <!-- Customer Interface -->
    <div v-if="currentRole === 'customer' && authToken" class="role-interface">
      <div class="tab-buttons">
        <button @click="customerTab = 'orders'" :class="{ active: customerTab === 'orders' }">Orders</button>
        <button @click="customerTab = 'suppliers'" :class="{ active: customerTab === 'suppliers' }">Suppliers</button>
        <button @click="customerTab = 'recommendations'" :class="{ active: customerTab === 'recommendations' }">Recommendations</button>
      </div>

      <!-- Orders Management -->
      <div v-if="customerTab === 'orders'" class="tab-content">
        <div class="endpoint">
          <h3>Get My Orders</h3>
          <button @click="customerGetOrders" class="action-button">Get Orders</button>
          <div class="response">{{ customerOrdersResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Create Order</h3>
          <div class="form-group">
            <label>Product:</label>
            <input v-model="customerOrderProduct">
          </div>
          <div class="form-group">
            <label>Quantity:</label>
            <input v-model="customerOrderQuantity" type="number">
          </div>
          <div class="form-group">
            <label>Delivery Date:</label>
            <input v-model="customerOrderDeliveryDate" type="date">
          </div>
          <button @click="customerCreateOrder" class="action-button">Create Order</button>
          <div class="response">{{ customerCreateOrderResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Update Order</h3>
          <div class="form-group">
            <label>Order ID:</label>
            <input v-model="customerUpdateOrderId" type="number">
          </div>
          <div class="form-group">
            <label>Update Data:</label>
            <textarea v-model="customerUpdateOrderData" placeholder='{"quantity": 3}'></textarea>
          </div>
          <button @click="customerUpdateOrder" class="action-button">Update Order</button>
          <div class="response">{{ customerUpdateOrderResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Delete Order</h3>
          <div class="form-group">
            <label>Order ID:</label>
            <input v-model="customerDeleteOrderId" type="number">
          </div>
          <button @click="customerDeleteOrder" class="action-button delete">Delete Order</button>
          <div class="response">{{ customerDeleteOrderResponse }}</div>
        </div>
      </div>

      <!-- Suppliers View -->
      <div v-if="customerTab === 'suppliers'" class="tab-content">
        <div class="endpoint">
          <h3>Find Suppliers</h3>
          <div class="form-group">
            <label>Category:</label>
            <input v-model="customerSupplierCategory">
          </div>
          <button @click="customerGetSuppliers" class="action-button">Search</button>
          <div class="response">{{ customerSuppliersResponse }}</div>
        </div>
      </div>

      <!-- Recommendations -->
      <div v-if="customerTab === 'recommendations'" class="tab-content">
        <div class="endpoint">
          <h3>Get Order Recommendations</h3>
          <div class="form-group">
            <label>Order ID:</label>
            <input v-model="customerRecommendationOrderId" type="number">
          </div>
          <button @click="customerGetOrderRecommendations" class="action-button">Get Recommendations</button>
          <div class="response">{{ customerOrderRecommendationsResponse }}</div>
        </div>

        <div class="endpoint">
          <h3>Get General Recommendations</h3>
          <div class="form-group">
            <label>Product Category:</label>
            <input v-model="customerRecommendationCategory">
          </div>
          <div class="form-group">
            <label>Top Only:</label>
            <input v-model="customerRecommendationTopOnly" type="checkbox">
          </div>
          <button @click="customerGetGeneralRecommendations" class="action-button">Get Recommendations</button>
          <div class="response">{{ customerGeneralRecommendationsResponse }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        // Role selection
        currentRole: 'customer',
        
        // Authentication
        authToken: null,
        authTab: 'login',
        loginEmail: '',
        loginPassword: '',
        authResponse: '',
        
        // Registration
        registerName: '',
        registerEmail: '',
        registerPassword: '',
        registerPassword2: '',
        registerRole: 'customer',
        registerCategories: '',
        registerResponse: '',
        
        adminUpdateUserId: '',
        adminUpdateUserData: '',
        adminUpdateUserResponse: '',
        adminDeleteUserId: '',
        adminDeleteUserResponse: '',
        adminCreateOrderData: '',
        adminCreateOrderResponse: '',
        adminUpdateOrderId: '',
        adminUpdateOrderData: '',
        adminUpdateOrderResponse: '',
        adminDeleteOrderId: '',
        adminDeleteOrderResponse: '',
        adminCreateSupplierData: '',
        adminCreateSupplierResponse: '',
        adminUpdateSupplierId: '',
        adminUpdateSupplierData: '',
        adminUpdateSupplierResponse: '',
        adminDeleteSupplierId: '',
        adminDeleteSupplierResponse: '',
        adminRecommendationOrderId: '',
        adminRecommendationAlgorithm: 'weighted',
        adminRecommendationsResponse: '',

        // Supplier data
        supplierRegisterName: '',
        supplierRegisterEmail: '',
        supplierRegisterPassword: '',
        supplierRegisterCategories: '',
        supplierRegisterResponse: '',

        // Customer data
        customerUpdateOrderId: '',
        customerUpdateOrderData: '',
        customerUpdateOrderResponse: '',
        customerDeleteOrderId: '',
        customerDeleteOrderResponse: '',
        customerCreateOrderResponse: '',
        customerRecommendationOrderId: '',
        customerOrderRecommendationsResponse: '',
        customerRecommendationCategory: '',
        customerRecommendationTopOnly: false,
        customerGeneralRecommendationsResponse: ''
      }
    },

    methods: {
      setRole(role) {
        this.currentRole = role;
      },
      
      async login() {
        try {
          const response = await fetch('http://localhost:8000/api/auth/login/', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              email: this.loginEmail,
              password: this.loginPassword
            })
          });
          
          const data = await response.json();
          if (response.ok) {
            this.authToken = data.access;
            this.authResponse = 'Login successful!';
          } else {
            this.authResponse = `Error: ${data.detail || 'Login failed'}`;
          }
        } catch (error) {
          this.authResponse = `Error: ${error.message}`;
        }
      },
      
      async register() {
        if (this.registerPassword !== this.registerPassword2) {
          this.registerResponse = 'Error: Passwords do not match';
          return;
        }
        
        try {
          const endpoint = this.registerRole === 'supplier' 
            ? '/api/suppliers/' 
            : '/api/auth/register/';
            
          const body = {
            name: this.registerName,
            email: this.registerEmail,
            password: this.registerPassword
          };
          
          if (this.registerRole === 'supplier') {
            body.categories = this.registerCategories.split(',').map(c => c.trim());
          }
          
          const response = await fetch(`http://localhost:8000${endpoint}`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(body)
          });
          
          const data = await response.json();
          this.registerResponse = response.ok 
            ? 'Registration successful! You can now log in.'
            : `Error: ${data.detail || 'Registration failed'}`;
        } catch (error) {
          this.registerResponse = `Error: ${error.message}`;
        }
      },
      
      async adminUpdateUser() {
        this.adminUpdateUserResponse = await this.makeRequest(
          'PUT',
          `/api/users/${this.adminUpdateUserId}/`,
          JSON.parse(this.adminUpdateUserData || '{}')
        );
      },
      async adminDeleteUser() {
        this.adminDeleteUserResponse = await this.makeRequest(
          'DELETE',
          `/api/users/${this.adminDeleteUserId}/`
        );
      },
      async adminCreateOrder() {
        this.adminCreateOrderResponse = await this.makeRequest(
          'POST',
          '/api/orders/',
          JSON.parse(this.adminCreateOrderData || '{}')
        );
      },
      async adminUpdateOrder() {
        this.adminUpdateOrderResponse = await this.makeRequest(
          'PUT',
          `/api/orders/${this.adminUpdateOrderId}/`,
          JSON.parse(this.adminUpdateOrderData || '{}')
        );
      },
      async adminDeleteOrder() {
        this.adminDeleteOrderResponse = await this.makeRequest(
          'DELETE',
          `/api/orders/${this.adminDeleteOrderId}/`
        );
      },
      async adminCreateSupplier() {
        this.adminCreateSupplierResponse = await this.makeRequest(
          'POST',
          '/api/suppliers/',
          JSON.parse(this.adminCreateSupplierData || '{}')
        );
      },
      async adminUpdateSupplier() {
        this.adminUpdateSupplierResponse = await this.makeRequest(
          'PUT',
          `/api/suppliers/${this.adminUpdateSupplierId}/`,
          JSON.parse(this.adminUpdateSupplierData || '{}')
        );
      },
      async adminDeleteSupplier() {
        this.adminDeleteSupplierResponse = await this.makeRequest(
          'DELETE',
          `/api/suppliers/${this.adminDeleteSupplierId}/`
        );
      },
      async adminGetRecommendations() {
        this.adminRecommendationsResponse = await this.makeRequest(
          'POST',
          '/api/recommendation',
          {
            order_id: this.adminRecommendationOrderId,
            algorithm: this.adminRecommendationAlgorithm
          }
        );
      },

      // Supplier methods
      async supplierRegister() {
        this.supplierRegisterResponse = await this.makeRequest(
          'POST',
          '/api/suppliers/',
          {
            name: this.supplierRegisterName,
            email: this.supplierRegisterEmail,
            password: this.supplierRegisterPassword,
            categories: this.supplierRegisterCategories.split(',').map(c => c.trim())
          },
          false // No auth needed for registration
        );
      },

      // Customer methods
      async customerUpdateOrder() {
        this.customerUpdateOrderResponse = await this.makeRequest(
          'PUT',
          `/api/orders/${this.customerUpdateOrderId}/`,
          JSON.parse(this.customerUpdateOrderData || '{}')
        );
      },
      async customerDeleteOrder() {
        this.customerDeleteOrderResponse = await this.makeRequest(
          'DELETE',
          `/api/orders/${this.customerDeleteOrderId}/`
        );
      },
      async customerGetOrderRecommendations() {
        this.customerOrderRecommendationsResponse = await this.makeRequest(
          'GET',
          `/api/orders/${this.customerRecommendationOrderId}/recommend_suppliers/`
        );
      },
      async customerGetGeneralRecommendations() {
        const endpoint = this.customerRecommendationTopOnly 
          ? '/api/recomendation?top_only=true' 
          : '/api/recomendation';
        
        this.customerGeneralRecommendationsResponse = await this.makeRequest(
          'POST',
          endpoint,
          {
            product_category: this.customerRecommendationCategory
          }
        );
      },

      // Enhanced makeRequest with error handling
      async makeRequest(method, endpoint, body = null, useAuth = true) {
        try {
          const headers = {
            'Content-Type': 'application/json'
          };
          
          if (useAuth && this.authToken) {
            headers['Authorization'] = `Bearer ${this.authToken}`;
          }

          const options = {
            method,
            headers
          };

          if (body) {
            options.body = typeof body === 'string' ? body : JSON.stringify(body);
          }

          const response = await fetch(`http://localhost:8000${endpoint}`, options);
          
          if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.detail || `HTTP error! status: ${response.status}`);
          }

          return await response.json();
        } catch (error) {
          console.error('API Error:', error);
          return { error: error.message };
        }
      }
    }
  }
</script>

<style>
.api-interface {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.role-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.role-selector button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  background: #e0e0e0;
}

.role-selector button.active {
  background: #42b983;
  color: white;
}

.auth-section {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.auth-tabs {
  display: flex;
  margin-bottom: 15px;
}

.auth-tabs button {
  padding: 10px 20px;
  border: none;
  background: #e0e0e0;
  cursor: pointer;
}

.auth-tabs button.active {
  background: #42b983;
  color: white;
}

.form-group {
  margin-bottom: 15px;
}

.form-row {
  display: flex;
  gap: 15px;
}

.form-row .form-group {
  flex: 1;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

input, select, textarea {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

textarea {
  min-height: 100px;
}

.auth-button, .action-button {
  background: #42b983;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

.tab-buttons {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.tab-buttons button {
  padding: 10px 20px;
  border: none;
  background: #e0e0e0;
  cursor: pointer;
}

.tab-buttons button.active {
  background: #42b983;
  color: white;
}

.tab-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.endpoint {
  margin-bottom: 30px;
}

.response {
  margin-top: 15px;
  padding: 10px;
  background: #f9f9f9;
  border-radius: 4px;
  font-family: monospace;
  white-space: pre-wrap;
}

.action-button.delete {
  background-color: #f44336;
}

.action-button.delete:hover {
  background-color: #d32f2f;
}

.endpoint {
  margin-bottom: 30px;
  padding: 15px;
  border: 1px solid #e0e0e0;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.endpoint h3 {
  margin-top: 0;
  color: #2c3e50;
}

</style>
