<template>
  <div class="api-interface">
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

    <!-- Role Selection -->
    <div class="role-selector">
      <button @click="setRole('admin')" :class="{ active: currentRole === 'admin' }">Admin</button>
      <button @click="setRole('supplier')" :class="{ active: currentRole === 'supplier' }">Supplier</button>
      <button @click="setRole('customer')" :class="{ active: currentRole === 'customer' }">Customer</button>
    </div>

    <!-- Admin Interface -->
    <div v-if="currentRole === 'admin'" class="role-interface">
      <div class="tab-buttons">
        <button @click="adminTab = 'users'" :class="{ active: adminTab === 'users' }">Users</button>
        <button @click="adminTab = 'orders'" :class="{ active: adminTab === 'orders' }">Orders</button>
        <button @click="adminTab = 'suppliers'" :class="{ active: adminTab === 'suppliers' }">Suppliers</button>
        <button @click="adminTab = 'recommendations'" :class="{ active: adminTab === 'recommendations' }">Recommendations</button>
      </div>

      <!-- User Management -->
      <div v-if="adminTab === 'users'" class="tab-content">
        <!-- Get All Users -->
        <div class="endpoint">
          <h3>Get All Users</h3>
          <div class="form-group">
            <label>Filter by Role:</label>
            <select v-model="adminUsersFilter">
              <option value="">All</option>
              <option value="admin">Admin</option>
              <option value="supplier">Supplier</option>
              <option value="customer">Customer</option>
            </select>
          </div>
          <button @click="adminGetUsers" class="action-button">Get Users</button>
          <div class="response">
            <pre>{{ adminUsersResponse }}</pre>
          </div>
          <button @click="adminGetUsers" class="action-button">Search Users</button>
          <div class="response">
            <table class="data-table" v-if="adminUsersResponse.results">
              <thead>
                <tr>
                  <th @click="sortUsers('id')">ID</th>
                  <th @click="sortUsers('email')">Email</th>
                  <th @click="sortUsers('name')">Name</th>
                  <th @click="sortUsers('role')">Role</th>
                  <th>Actions</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="user in adminUsersResponse.results" :key="user.id">
                  <td>{{ user.id }}</td>
                  <td>{{ user.email }}</td>
                  <td>{{ user.name }}</td>
                  <td>{{ user.role }}</td>
                  <td>
                    <button @click="loadUserForEdit(user)" class="small-button">Edit</button>
                  </td>
                </tr>
              </tbody>
            </table>
            <pre v-else>{{ adminUsersResponse }}</pre>
          </div>
        </div>

        <!-- Update User -->
        <div class="endpoint" v-if="currentEditUser">
          <h3>Edit User: {{ currentEditUser.email }}</h3>
          <div class="form-row">
            <div class="form-group">
              <label>Email:</label>
              <input v-model="currentEditUser.email" type="email">
            </div>
            <div class="form-group">
              <label>Name:</label>
              <input v-model="currentEditUser.name">
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Role:</label>
              <select v-model="currentEditUser.role">
                <option value="admin">Admin</option>
                <option value="supplier">Supplier</option>
                <option value="customer">Customer</option>
              </select>
            </div>
            <div class="form-group">
              <label>Status:</label>
              <div class="checkbox-group">
                <label><input v-model="currentEditUser.is_active" type="checkbox"> Active</label>
                <label><input v-model="currentEditUser.is_staff" type="checkbox"> Staff</label>
                <label><input v-model="currentEditUser.is_superuser" type="checkbox"> Superuser</label>
              </div>
            </div>
          </div>
          <div class="button-group">
            <button @click="adminUpdateUser" class="action-button">Save Changes</button>
            <button @click="cancelEdit" class="action-button secondary">Cancel</button>
          </div>
          <div class="response">{{ adminUpdateUserResponse }}</div>
        </div>

        <!-- Delete User -->
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
        <!-- Order Filters -->
        <div class="endpoint">
          <h3>Order Management</h3>
          <div class="form-row">
            <div class="form-group">
              <label>Status:</label>
              <select v-model="adminOrdersFilter.status">
                <option value="">All Statuses</option>
                <option value="pending">Pending</option>
                <option value="completed">Completed</option>
              </select>
            </div>
            <div class="form-group">
              <label>Customer ID:</label>
              <input v-model="adminOrdersFilter.customer_id" type="number" placeholder="Filter by customer">
            </div>
            <div class="form-group">
              <label>OKPD2:</label>
              <select v-model="adminOrdersFilter.okpd2">
                <option value="">All Categories</option>
                <option value="29.10">29.10</option>
                <option value="29.20">29.20</option>
                <option value="29.31">29.31</option>
                <option value="29.32">29.32</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Delivery Region:</label>
              <select v-model="adminOrdersFilter.delivery_region">
                <option value="">All Regions</option>
                <option v-for="region in deliveryRegions" :value="region">{{ region }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>Law Type:</label>
              <select v-model="adminOrdersFilter.law_type">
                <option value="">All Types</option>
                <option value="44_FZ">44-ФЗ</option>
                <option value="223_FZ">223-ФЗ</option>
              </select>
            </div>
          </div>
          <button @click="adminGetOrders" class="action-button">Search Orders</button>
          
          <!-- Orders Table -->
          <div class="response">
            <table class="data-table" v-if="adminOrdersResponse.results">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Customer</th>
                  <th>OKPD2</th>
                  <th>Amount</th>
                  <th>Region</th>
                  <th>Status</th>
                  <th>Actions</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="order in adminOrdersResponse.results" :key="order.id">
                  <td>{{ order.id }}</td>
                  <td>{{ order.customer.email }} (ID: {{ order.customer.id }})</td>
                  <td>{{ order.okpd2 }}</td>
                  <td>{{ order.contract_amount }}</td>
                  <td>{{ order.delivery_region }}</td>
                  <td>{{ order.status }}</td>
                  <td>
                    <button @click="loadOrderForEdit(order)" class="small-button">Edit</button>
                  </td>
                </tr>
              </tbody>
            </table>
            <pre v-else>{{ adminOrdersResponse }}</pre>
          </div>
        </div>

        <!-- Order Edit Form (Pre-filled) -->
        <div class="endpoint" v-if="currentEditOrder">
          <h3>Edit Order #{{ currentEditOrder.id }}</h3>
          <div class="form-row">
            <div class="form-group">
              <label>Customer ID:</label>
              <input v-model="currentEditOrder.customer.id" type="number" disabled>
            </div>
            <div class="form-group">
              <label>OKPD2:</label>
              <select v-model="currentEditOrder.okpd2">
                <option value="29.10">29.10</option>
                <option value="29.20">29.20</option>
                <option value="29.31">29.31</option>
                <option value="29.32">29.32</option>
              </select>
            </div>
          </div>
          <div class="form-group">
            <label>Description:</label>
            <textarea v-model="currentEditOrder.description"></textarea>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Contract Amount:</label>
              <input v-model="currentEditOrder.contract_amount" type="number" step="0.01">
            </div>
            <div class="form-group">
              <label>Delivery Region:</label>
              <select v-model="currentEditOrder.delivery_region">
                <option v-for="region in deliveryRegions" :value="region">{{ region }}</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Law Type:</label>
              <select v-model="currentEditOrder.law_type">
                <option value="44_FZ">44-ФЗ</option>
                <option value="223_FZ">223-ФЗ</option>
              </select>
            </div>
            <div class="form-group">
              <label>Status:</label>
              <select v-model="currentEditOrder.status">
                <option value="pending">Pending</option>
                <option value="completed">Completed</option>
              </select>
            </div>
          </div>
          <div class="button-group">
            <button @click="adminUpdateOrder" class="action-button">Save Changes</button>
            <button @click="cancelEdit" class="action-button secondary">Cancel</button>
          </div>
          <div class="response">{{ adminUpdateOrderResponse }}</div>
        </div>

        <!-- Delete Order -->
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
        <!-- Get All Suppliers -->
        <div class="endpoint">
          <h3>Get All Suppliers</h3>
          <div class="form-row">
            <div class="form-group">
              <label>Verification:</label>
              <select v-model="adminSuppliersFilter.is_verified">
                <option value="">All</option>
                <option value="true">Verified</option>
                <option value="false">Unverified</option>
              </select>
            </div>
            <div class="form-group">
              <label>OKFS:</label>
              <select v-model="adminSuppliersFilter.okfs">
                <option value="">All OKFS</option>
                <option v-for="(label, value) in okfsChoices" :value="value">{{ label }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>OKOPF:</label>
              <select v-model="adminSuppliersFilter.okopf">
                <option value="">All OKOPF</option>
                <option v-for="(label, value) in okopfChoices" :value="value">{{ label }}</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Search:</label>
              <input v-model="adminSuppliersFilter.search" placeholder="Name, INN, or OGRN">
            </div>
            <div class="form-group">
              <label>Risk Level:</label>
              <select v-model="adminSuppliersFilter.risk_level">
                <option value="">All</option>
                <option value="low">Low</option>
                <option value="medium">Medium</option>
                <option value="high">High</option>
              </select>
            </div>
          </div>
          <button @click="adminGetSuppliers" class="action-button">Get Suppliers</button>
          <div class="response">
            <table class="data-table" v-if="adminSuppliersResponse.results">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>Name</th>
                  <th>INN</th>
                  <th>OKFS</th>
                  <th>Risk</th>
                  <th>Verified</th>
                  <th>Actions</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="supplier in adminSuppliersResponse.results" :key="supplier.id">
                  <td>{{ supplier.id }}</td>
                  <td>{{ supplier.full_name }}</td>
                  <td>{{ supplier.inn }}</td>
                  <td>{{ supplier.okfs }}</td>
                  <td :class="`risk-${supplier.index_due_diligence_word?.toLowerCase() || 'unknown'}`">
                    {{ supplier.index_due_diligence_word || 'N/A' }}
                  </td>
                  <td>{{ supplier.is_verified ? '✓' : '✗' }}</td>
                  <td>
                    <button @click="loadSupplierForEdit(supplier)" class="small-button">Edit</button>
                  </td>
                </tr>
              </tbody>
            </table>
            <pre v-else>{{ adminSuppliersResponse }}</pre>
          </div>
        </div>

        <!-- Update Supplier -->
        <div class="endpoint" v-if="currentEditSupplier">
          <h3>Edit Supplier: {{ currentEditSupplier.full_name }}</h3>
          <div class="form-row">
            <div class="form-group">
              <label>Full Name:</label>
              <input v-model="currentEditSupplier.full_name">
            </div>
            <div class="form-group">
              <label>Short Name:</label>
              <input v-model="currentEditSupplier.short_name">
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>INN:</label>
              <input v-model="currentEditSupplier.inn" maxlength="12">
            </div>
            <div class="form-group">
              <label>OGRN:</label>
              <input v-model="currentEditSupplier.ogrn" maxlength="15">
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>OKFS:</label>
              <select v-model="currentEditSupplier.okfs">
                <option v-for="(label, value) in okfsChoices" :value="value">{{ label }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>OKOPF:</label>
              <select v-model="currentEditSupplier.okopf">
                <option v-for="(label, value) in okopfChoices" :value="value">{{ label }}</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Email:</label>
              <input v-model="currentEditSupplier.email" type="email">
            </div>
            <div class="form-group">
              <label>Leader:</label>
              <input v-model="currentEditSupplier.leader">
            </div>
          </div>
          <div class="form-group">
            <label>Categories (OKPD2):</label>
            <input v-model="currentEditSupplier.okpd2_categories" placeholder="Comma-separated codes">
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Verified:</label>
              <input v-model="currentEditSupplier.is_verified" type="checkbox">
            </div>
            <div class="form-group">
              <label>Due Diligence Index:</label>
              <input v-model="currentEditSupplier.index_due_diligence" type="number" min="1" max="99">
              <span v-if="currentEditSupplier.index_due_diligence">
                ({{ currentEditSupplier.index_due_diligence_word }})
              </span>
            </div>
          </div>
          <div class="button-group">
            <button @click="adminUpdateSupplier" class="action-button">Save Changes</button>
            <button @click="cancelEdit" class="action-button secondary">Cancel</button>
            <button @click="adminDeleteSupplier" class="action-button delete" 
                    v-if="currentEditSupplier.id">Delete Supplier</button>
          </div>
          <div class="response">{{ adminUpdateSupplierResponse }}</div>
        </div>

        <!-- Create Supplier -->
        <div class="endpoint">
          <h3>Create Supplier</h3>
          <div class="form-row">
            <div class="form-group">
              <label>User ID:</label>
              <input v-model="adminCreateSupplierUserId" type="number">
            </div>
            <div class="form-group">
              <label>Full Name:</label>
              <input v-model="adminCreateSupplierFullName">
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>INN:</label>
              <input v-model="adminCreateSupplierInn" maxlength="12">
            </div>
            <div class="form-group">
              <label>OGRN:</label>
              <input v-model="adminCreateSupplierOgrn" maxlength="15">
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>OKFS:</label>
              <select v-model="adminCreateSupplierOkfs">
                <option v-for="(label, value) in okfsChoices" :value="value">{{ label }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>OKOPF:</label>
              <select v-model="adminCreateSupplierOkopf">
                <option v-for="(label, value) in okopfChoices" :value="value">{{ label }}</option>
              </select>
            </div>
          </div>
          <div class="form-group">
            <label>OKPD2 Categories (comma-separated):</label>
            <input v-model="adminCreateSupplierOkpd2">
          </div>
          <button @click="adminCreateSupplier" class="action-button">Create Supplier</button>
          <div class="response">{{ adminCreateSupplierResponse }}</div>
        </div>

        <!-- Delete Supplier -->
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
          <div class="form-group">
            <label>Top Only:</label>
            <input v-model="adminRecommendationTopOnly" type="checkbox">
          </div>
          <button @click="adminGetRecommendations" class="action-button">Get Recommendations</button>
          <div class="response">
            <pre>{{ adminRecommendationsResponse }}</pre>
          </div>
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

    <div v-if="currentRole === 'customer' && authToken" class="role-interface">
      <div class="tab-buttons">
        <button @click="customerTab = 'orders'" :class="{ active: customerTab === 'orders' }">Orders</button>
        <button @click="customerTab = 'suppliers'" :class="{ active: customerTab === 'suppliers' }">Suppliers</button>
        <button @click="customerTab = 'recommendations'" :class="{ active: customerTab === 'recommendations' }">Recommendations</button>
      </div>

      <!-- Orders Management -->
      <div v-if="customerTab === 'orders'" class="tab-content">
        <!-- Get All Orders -->
        <div class="endpoint">
          <h3>Get My Orders</h3>
          <div class="form-row">
            <div class="form-group">
              <label>Status:</label>
              <select v-model="customerOrdersFilter.status">
                <option value="">All</option>
                <option value="pending">Pending</option>
                <option value="completed">Completed</option>
              </select>
            </div>
            <div class="form-group">
              <label>OKPD2:</label>
              <select v-model="customerOrdersFilter.okpd2">
                <option value="">All</option>
                <option value="29.10">29.10</option>
                <option value="29.20">29.20</option>
                <option value="29.31">29.31</option>
                <option value="29.32">29.32</option>
              </select>
            </div>
          </div>
          <button @click="customerGetOrders" class="action-button">Get Orders</button>
          <div class="response">
            <table class="data-table" v-if="customerOrdersResponse.results">
              <thead>
                <tr>
                  <th>ID</th>
                  <th>OKPD2</th>
                  <th>Description</th>
                  <th>Amount</th>
                  <th>Region</th>
                  <th>Law Type</th>
                  <th>Actions</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="order in customerOrdersResponse.results" :key="order.id">
                  <td>{{ order.id }}</td>
                  <td>{{ order.okpd2 }}</td>
                  <td>{{ order.description }}</td>
                  <td>{{ order.contract_amount }}</td>
                  <td>{{ order.delivery_region }}</td>
                  <td>{{ order.law_type }}</td>
                  <td>
                    <button @click="loadCustomerOrderForEdit(order)" class="small-button">Edit</button>
                    <button @click="confirmDeleteOrder(order.id)" class="small-button delete">Delete</button>
                  </td>
                </tr>
              </tbody>
            </table>
            <pre v-else>{{ customerOrdersResponse }}</pre>
          </div>
        </div>

        <!-- Create Order -->
        <div class="endpoint">
          <h3>Create Order</h3>
          <div class="form-row">
            <div class="form-group">
              <label>OKPD2:</label>
              <select v-model="customerCreateOrderData.okpd2">
                <option value="29.10">29.10</option>
                <option value="29.20">29.20</option>
                <option value="29.31">29.31</option>
                <option value="29.32">29.32</option>
              </select>
            </div>
            <div class="form-group">
              <label>Law Type:</label>
              <select v-model="customerCreateOrderData.law_type">
                <option value="44_FZ">44-ФЗ</option>
                <option value="223_FZ">223-ФЗ</option>
              </select>
            </div>
          </div>
          <div class="form-group">
            <label>Description:</label>
            <textarea v-model="customerCreateOrderData.description" placeholder="Detailed order description"></textarea>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Contract Amount:</label>
              <input v-model="customerCreateOrderData.contract_amount" type="number" step="0.01" placeholder="0.00">
            </div>
            <div class="form-group">
              <label>Delivery Region:</label>
              <select v-model="customerCreateOrderData.delivery_region">
                <option v-for="region in deliveryRegions" :value="region">{{ region }}</option>
              </select>
            </div>
          </div>
          <button @click="customerCreateOrder" class="action-button">Create Order</button>
          <div class="response">{{ customerCreateOrderResponse }}</div>
        </div>

        <!-- Edit Order -->
        <div class="endpoint" v-if="customerCurrentEditOrder">
          <h3>Edit Order #{{ customerCurrentEditOrder.id }}</h3>
          <div class="form-row">
            <div class="form-group">
              <label>OKPD2:</label>
              <select v-model="customerCurrentEditOrder.okpd2">
                <option value="29.10">29.10</option>
                <option value="29.20">29.20</option>
                <option value="29.31">29.31</option>
                <option value="29.32">29.32</option>
              </select>
            </div>
            <div class="form-group">
              <label>Law Type:</label>
              <select v-model="customerCurrentEditOrder.law_type">
                <option value="44_FZ">44-ФЗ</option>
                <option value="223_FZ">223-ФЗ</option>
              </select>
            </div>
          </div>
          <div class="form-group">
            <label>Description:</label>
            <textarea v-model="customerCurrentEditOrder.description"></textarea>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Contract Amount:</label>
              <input v-model="customerCurrentEditOrder.contract_amount" type="number" step="0.01">
            </div>
            <div class="form-group">
              <label>Delivery Region:</label>
              <select v-model="customerCurrentEditOrder.delivery_region">
                <option v-for="region in deliveryRegions" :value="region">{{ region }}</option>
              </select>
            </div>
          </div>
          <div class="button-group">
            <button @click="customerUpdateOrder" class="action-button">Save Changes</button>
            <button @click="cancelCustomerEdit" class="action-button secondary">Cancel</button>
          </div>
          <div class="response">{{ customerUpdateOrderResponse }}</div>
        </div>
      </div>

      <!-- Suppliers View -->
      <div v-if="customerTab === 'suppliers'" class="tab-content">
        <!-- Get All Suppliers -->
        <div class="endpoint">
          <h3>Find Suppliers</h3>
          <div class="form-row">
            <div class="form-group">
              <label>OKPD2:</label>
              <select v-model="customerSuppliersFilter.okpd2">
                <option value="">All</option>
                <option value="29.10">29.10</option>
                <option value="29.20">29.20</option>
                <option value="29.31">29.31</option>
                <option value="29.32">29.32</option>
              </select>
            </div>
            <div class="form-group">
              <label>OKFS:</label>
              <select v-model="customerSuppliersFilter.okfs">
                <option value="">All</option>
                <option v-for="(label, value) in okfsChoices" :value="value">{{ label }}</option>
              </select>
            </div>
            <div class="form-group">
              <label>OKOPF:</label>
              <select v-model="customerSuppliersFilter.okopf">
                <option value="">All</option>
                <option v-for="(label, value) in okopfChoices" :value="value">{{ label }}</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Risk Level:</label>
              <select v-model="customerSuppliersFilter.risk_level">
                <option value="">All</option>
                <option value="low">Low</option>
                <option value="medium">Medium</option>
                <option value="high">High</option>
              </select>
            </div>
            <div class="form-group">
              <label>Search:</label>
              <input v-model="customerSuppliersFilter.search" placeholder="Name, INN, or OGRN">
            </div>
          </div>
          <button @click="customerGetSuppliers" class="action-button">Search Suppliers</button>
          <div class="response">
            <table class="data-table" v-if="customerSuppliersResponse.results">
              <thead>
                <tr>
                  <th>Name</th>
                  <th>INN</th>
                  <th>OKFS</th>
                  <th>Risk Level</th>
                  <th>Categories</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="supplier in customerSuppliersResponse.results" :key="supplier.id">
                  <td>{{ supplier.full_name }}</td>
                  <td>{{ supplier.inn }}</td>
                  <td>{{ supplier.okfs }}</td>
                  <td :class="`risk-${supplier.index_due_diligence_word?.toLowerCase() || 'unknown'}`">
                    {{ supplier.index_due_diligence_word || 'N/A' }}
                  </td>
                  <td>{{ supplier.okpd2_categories?.join(', ') || 'N/A' }}</td>
                </tr>
              </tbody>
            </table>
            <pre v-else>{{ customerSuppliersResponse }}</pre>
          </div>
        </div>
      </div>

      <!-- Recommendations -->
      <div v-if="customerTab === 'recommendations'" class="tab-content">
        <!-- Order Recommendations -->
        <div class="endpoint">
          <h3>Get Order Recommendations</h3>
          <div class="form-group">
            <label>Order ID:</label>
            <input v-model="customerRecommendationOrderId" type="number">
          </div>
          <div class="form-group">
            <label>Top Only:</label>
            <input v-model="customerRecommendationTopOnly" type="checkbox">
          </div>
          <button @click="customerGetOrderRecommendations" class="action-button">Get Recommendations</button>
          <div class="response">
            <table class="data-table" v-if="customerOrderRecommendationsResponse.results">
              <thead>
                <tr>
                  <th>Supplier</th>
                  <th>INN</th>
                  <th>Score</th>
                  <th>Risk Level</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="rec in customerOrderRecommendationsResponse.results" :key="rec.supplier.id">
                  <td>{{ rec.supplier.full_name }}</td>
                  <td>{{ rec.supplier.inn }}</td>
                  <td>{{ rec.score.toFixed(2) }}</td>
                  <td :class="`risk-${rec.supplier.index_due_diligence_word?.toLowerCase() || 'unknown'}`">
                    {{ rec.supplier.index_due_diligence_word || 'N/A' }}
                  </td>
                </tr>
              </tbody>
            </table>
            <pre v-else>{{ customerOrderRecommendationsResponse }}</pre>
          </div>
        </div>

        <!-- General Recommendations -->
        <div class="endpoint">
          <h3>Get General Recommendations</h3>
          <div class="form-row">
            <div class="form-group">
              <label>OKPD2:</label>
              <select v-model="customerGeneralRecommendationData.okpd2">
                <option value="29.10">29.10</option>
                <option value="29.20">29.20</option>
                <option value="29.31">29.31</option>
                <option value="29.32">29.32</option>
              </select>
            </div>
            <div class="form-group">
              <label>Region:</label>
              <select v-model="customerGeneralRecommendationData.region">
                <option v-for="region in deliveryRegions" :value="region">{{ region }}</option>
              </select>
            </div>
          </div>
          <div class="form-row">
            <div class="form-group">
              <label>Law Type:</label>
              <select v-model="customerGeneralRecommendationData.law_type">
                <option value="44_FZ">44-ФЗ</option>
                <option value="223_FZ">223-ФЗ</option>
              </select>
            </div>
            <div class="form-group">
              <label>Top Only:</label>
              <input v-model="customerGeneralRecommendationData.top_only" type="checkbox">
            </div>
          </div>
          <button @click="customerGetGeneralRecommendations" class="action-button">Get Recommendations</button>
          <div class="response">
            <table class="data-table" v-if="customerGeneralRecommendationsResponse.results">
              <thead>
                <tr>
                  <th>Supplier</th>
                  <th>INN</th>
                  <th>Score</th>
                  <th>Risk Level</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="rec in customerGeneralRecommendationsResponse.results" :key="rec.supplier.id">
                  <td>{{ rec.supplier.full_name }}</td>
                  <td>{{ rec.supplier.inn }}</td>
                  <td>{{ rec.score.toFixed(2) }}</td>
                  <td :class="`risk-${rec.supplier.index_due_diligence_word?.toLowerCase() || 'unknown'}`">
                    {{ rec.supplier.index_due_diligence_word || 'N/A' }}
                  </td>
                </tr>
              </tbody>
            </table>
            <pre v-else>{{ customerGeneralRecommendationsResponse }}</pre>
          </div>
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
        
        // Users Management
        adminUsersFilter: {
          role: '',
          search: '',
          is_active: '',
          ordering: 'id'
        },
        adminUsersResponse: {},
        currentEditUser: null,
        adminDeleteUserId: '',
        adminDeleteUserResponse: '',

        // Orders Management
        adminOrdersFilter: {
          status: '',
          customer_id: '',
          okpd2: '',
          delivery_region: '',
          law_type: '',
          ordering: 'id'
        },
        adminCreateOrderCustomerId: '',
        adminCreateOrderOkpd2: '29.10',
        adminCreateOrderDescription: '',
        adminCreateOrderContractAmount: 0,
        adminCreateOrderDeliveryRegion: 'Москва',
        adminCreateOrderLawType: '44_FZ',
        adminCreateOrderResponse: '',
        adminOrdersResponse: {},
        currentEditOrder: null,
        adminDeleteOrderId: '',
        adminDeleteOrderResponse: '',

        // Suppliers Management
        adminSuppliersFilter: {
          is_verified: '',
          okfs: '',
          okopf: '',
          search: '',
          risk_level: ''
        },
        adminSuppliersResponse: '',
        adminCreateSupplierUserId: '',
        adminCreateSupplierFullName: '',
        adminCreateSupplierInn: '',
        adminCreateSupplierOgrn: '',
        adminCreateSupplierOkfs: 'Частная собственность',
        adminCreateSupplierOkopf: 'Общества с ограниченной ответственностью',
        adminCreateSupplierOkpd2: '',
        adminCreateSupplierResponse: '',
        currentEditSupplier: null,
        adminSupplierResponse: {},
        adminDeleteSupplierId: '',
        adminDeleteSupplierResponse: '',

        // Recommendations
        adminRecommendationOrderId: '',
        adminRecommendationAlgorithm: 'weighted',
        adminRecommendationTopOnly: false,
        adminRecommendationsResponse: '',

        // Constants from models
        deliveryRegions: [
          "Алтайский край", "Амурская область", "Архангельская область", "Архангельская область", "Байконур", "Белгородская область", 
          "Брянская область", "Владимирская область", "Волгоградская область", "Вологодская область", "Воронежская область", 
          "Донецкая Народная Республика", "Еврейская автономная область", "Забайкальский край", "Запорожская область", "Ивановская область", 
          "Иркутская область", "Кабардино-Балкарская Республика", "Калининградская область", "Калужская область", "Камчатский край", 
          "Карачаево-Черкесская Республика", "Кемеровская область - Кузбасс", "Кировская область", "Костромская область", "Краснодарский край", 
          "Красноярский край", "Курганская область", "Курская область", "Ленинградская область", "Липецкая область", "Луганская Народная Республика", 
          "Магаданская область", "Москва", "Московская область", "Мурманская область", "Ненецкий автономный округ (Архангельская область)", 
          "Нижегородская область", "Новгородская область", "Новосибирская область", "Омская область", "Оренбургская область", "Орловская область", "Пензенская область", 
          "Пермский край", "Приморский край", "Псковская область", "Республика Адыгея (Адыгея)", "Республика Алтай", 
          "Республика Башкортостан", "Республика Бурятия", "Республика Дагестан", "Республика Ингушетия", "Республика Калмыкия", 
          "Республика Карелия", "Республика Коми", "Республика Крым", "Республика Марий Эл", "Республика Мордовия", 
          "Республика Саха (Якутия)", "Республика Северная Осетия-Алания", "Республика Татарстан (Татарстан)", 
          "Республика Тыва", "Республика Хакасия", "Ростовская область", "Рязанская область", "Самарская область", "Санкт-Петербург", 
          "Саратовская область", "Сахалинская область", "Свердловская область", 
          "Севастополь", "Смоленская область", "Ставропольский край", "Тамбовская область", 
          "Тверская область", "Томская область", "Тульская область", "Тюменская область", 
          "Удмуртская Республика", "Ульяновская область", "Хабаровский край", 
          "Ханты-Мансийский автономный округ - Югра (Тюменская область)", "Херсонская область", "Челябинская область", 
          "Чеченская Республика", "Чувашская Республика - Чувашия", "Чукотский автономный округ", 
          "Ямало-Ненецкий автономный округ (Тюменская область)", "Ярославская область"
        ],

        okfsChoices: {
          "Частная собственность": "Частная собственность",
          "Государственная собственность": "Государственная собственность",
          "Индивидуальные предприниматели": "Индивидуальные предприниматели",
          "Собственность иностранных граждан и лиц без гражданства": "Собственность иностранных граждан и лиц без гражданства",
          "Иная смешанная российская собственность": "Иная смешанная российская собственность", 
          "Совместная частная и иностранная собственность": "Совместная частная и иностранная собственность",
          "Федеральная собственность": "Федеральная собственность"
        },

        okopfChoices: {
          "Общества с ограниченной ответственностью": "Общества с ограниченной ответственностью",
          "Акционерные общества": "Акционерные общества", 
          "Индивидуальные предприниматели": "Индивидуальные предприниматели",
          "Закрытые акционерные общества": "Закрытые акционерные общества", 
          "Открытые акционерные общества": "Открытые акционерные общества",
          "Казенные учреждения": "Казенные учреждения"
        },

        // Supplier data
        supplierRegisterName: '',
        supplierRegisterEmail: '',
        supplierRegisterPassword: '',
        supplierRegisterCategories: '',
        supplierRegisterResponse: '',

        // Customer data
        customerTab: 'orders',
        customerOrdersFilter: {
          status: '',
          okpd2: ''
        },
        customerOrdersResponse: {},
        customerCurrentEditOrder: null,
        
        customerCreateOrderData: {
          okpd2: '29.10',
          description: '',
          contract_amount: 0,
          delivery_region: 'Москва',
          law_type: '44_FZ'
        },
        customerCreateOrderResponse: '',
        customerUpdateOrderResponse: '',
        
        customerSuppliersFilter: {
          okpd2: '',
          okfs: '',
          okopf: '',
          risk_level: '',
          search: ''
        },
        customerSuppliersResponse: {},
        
        customerRecommendationOrderId: '',
        customerRecommendationTopOnly: false,
        customerOrderRecommendationsResponse: {},
        
        customerGeneralRecommendationData: {
          okpd2: '29.10',
          region: 'Москва',
          law_type: '44_FZ',
          top_only: false
        },
        customerGeneralRecommendationsResponse: {}
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
      
      // Admin: Users
      async adminGetUsers() {
        const query = Object.entries(this.adminUsersFilter)
          .filter(([_, value]) => value !== '')
          .map(([key, value]) => `${key}=${value}`)
          .join('&');
        
        this.adminUsersResponse = await this.makeRequest('GET', `/users/?${query}`);
      },

      loadUserForEdit(user) {
        this.currentEditUser = JSON.parse(JSON.stringify(user));
      },

      async adminUpdateUser() {
        try {
          const response = await this.makeRequest(
            'PUT',
            `/users/${this.currentEditUser.id}/`,
            {
              email: this.currentEditUser.email,
              name: this.currentEditUser.name,
              role: this.currentEditUser.role,
              is_active: this.currentEditUser.is_active,
              is_staff: this.currentEditUser.is_staff,
              is_superuser: this.currentEditUser.is_superuser
            }
          );
          this.adminUpdateUserResponse = "User updated successfully!";
          await this.adminGetUsers();
        } catch (error) {
          this.adminUpdateUserResponse = `Error: ${error.message}`;
        }
      },

      async adminDeleteUser() {
        this.adminDeleteUserResponse = await this.makeRequest(
          'DELETE',
          `/users/${this.adminDeleteUserId}/`
        );
      },

      // Admin: Orders
      async adminGetOrders() {
        const query = Object.entries(this.adminOrdersFilter)
          .filter(([_, value]) => value !== '')
          .map(([key, value]) => `${key}=${value}`)
          .join('&');
        
        this.adminOrdersResponse = await this.makeRequest('GET', `/orders/?${query}`);
      },

      async adminCreateOrder() {
        this.adminCreateOrderResponse = await this.makeRequest(
          'POST',
          '/orders/',
          {
            customer: this.adminCreateOrderCustomerId,
            okpd2: this.adminCreateOrderOkpd2,
            description: this.adminCreateOrderDescription,
            contract_amount: this.adminCreateOrderContractAmount,
            delivery_region: this.adminCreateOrderDeliveryRegion,
            law_type: this.adminCreateOrderLawType
          }
        );
      },

      loadOrderForEdit(order) {
        this.currentEditOrder = JSON.parse(JSON.stringify(order));
      },
      
      async adminUpdateOrder() {
        try {
          const response = await this.makeRequest(
            'PUT',
            `/orders/${this.currentEditOrder.id}/`,
            {
              okpd2: this.currentEditOrder.okpd2,
              description: this.currentEditOrder.description,
              contract_amount: this.currentEditOrder.contract_amount,
              delivery_region: this.currentEditOrder.delivery_region,
              law_type: this.currentEditOrder.law_type,
              status: this.currentEditOrder.status
            }
          );
          this.adminUpdateOrderResponse = "Order updated successfully!";
          await this.adminGetOrders(); // Refresh the list
        } catch (error) {
          this.adminUpdateOrderResponse = `Error: ${error.message}`;
        }
      },

      async adminDeleteOrder() {
        this.adminDeleteOrderResponse = await this.makeRequest(
          'DELETE',
          `/orders/${this.adminDeleteOrderId}/`
        );
      },

      // Admin: Suppliers
      async adminGetSuppliers() {
        const query = Object.entries(this.adminSuppliersFilter)
          .filter(([_, value]) => value !== '')
          .map(([key, value]) => `${key}=${value}`)
          .join('&');
        
        this.adminSuppliersResponse = await this.makeRequest('GET', `/suppliers/?${query}`);
      },

      loadSupplierForEdit(supplier) {
        this.currentEditSupplier = JSON.parse(JSON.stringify(supplier));
      },

      async adminUpdateSupplier() {
        try {
          const response = await this.makeRequest(
            'PUT',
            `/suppliers/${this.currentEditSupplier.id}/`,
            {
              full_name: this.currentEditSupplier.full_name,
              short_name: this.currentEditSupplier.short_name,
              inn: this.currentEditSupplier.inn,
              ogrn: this.currentEditSupplier.ogrn,
              okfs: this.currentEditSupplier.okfs,
              okopf: this.currentEditSupplier.okopf,
              email: this.currentEditSupplier.email,
              leader: this.currentEditSupplier.leader,
              okpd2_categories: this.currentEditSupplier.okpd2_categories,
              is_verified: this.currentEditSupplier.is_verified,
              index_due_diligence: this.currentEditSupplier.index_due_diligence
            }
          );
          this.adminUpdateSupplierResponse = "Supplier updated successfully!";
          await this.adminGetSuppliers(); // Refresh the list
        } catch (error) {
          this.adminUpdateSupplierResponse = `Error: ${error.message}`;
        }
      },

      async adminDeleteSupplier() {
        this.adminDeleteSupplierResponse = await this.makeRequest(
          'DELETE',
          `/suppliers/${this.adminDeleteSupplierI}/`
        );
      },

      // Admin: Recommendations
      async adminGetRecommendations() {
        this.adminRecommendationsResponse = await this.makeRequest(
          'POST',
          '/recommendation',
          {
            order_id: this.adminRecommendationOrderId,
            algorithm: this.adminRecommendationAlgorithm,
            top_only: this.adminRecommendationTopOnly
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
      async customerGetOrders() {
        const query = Object.entries(this.customerOrdersFilter)
          .filter(([_, value]) => value !== '')
          .map(([key, value]) => `${key}=${value}`)
          .join('&');
        
        this.customerOrdersResponse = await this.makeRequest('GET', `/orders/?${query}`);
      },

      async customerCreateOrder() {
        try {
          this.customerCreateOrderResponse = await this.makeRequest(
            'POST',
            '/orders/',
            this.customerCreateOrderData
          );
          await this.customerGetOrders(); // Refresh the list
        } catch (error) {
          this.customerCreateOrderResponse = `Error: ${error.message}`;
        }
      },

      loadCustomerOrderForEdit(order) {
        this.customerCurrentEditOrder = JSON.parse(JSON.stringify(order));
      },

      cancelCustomerEdit() {
        this.customerCurrentEditOrder = null;
      },

      async customerUpdateOrder() {
        try {
          this.customerUpdateOrderResponse = await this.makeRequest(
            'PUT',
            `/orders/${this.customerCurrentEditOrder.id}/`,
            {
              okpd2: this.customerCurrentEditOrder.okpd2,
              description: this.customerCurrentEditOrder.description,
              contract_amount: this.customerCurrentEditOrder.contract_amount,
              delivery_region: this.customerCurrentEditOrder.delivery_region,
              law_type: this.customerCurrentEditOrder.law_type
            }
          );
          await this.customerGetOrders(); // Refresh the list
        } catch (error) {
          this.customerUpdateOrderResponse = `Error: ${error.message}`;
        }
      },

      async confirmDeleteOrder(orderId) {
        if (confirm('Are you sure you want to delete this order?')) {
          const response = await this.makeRequest('DELETE', `/orders/${orderId}/`);
          if (!response.error) {
            await this.customerGetOrders(); // Refresh the list
          }
        }
      },

      async customerGetSuppliers() {
        const query = Object.entries(this.customerSuppliersFilter)
          .filter(([_, value]) => value !== '')
          .map(([key, value]) => `${key}=${value}`)
          .join('&');
        
        this.customerSuppliersResponse = await this.makeRequest('GET', `/suppliers/?${query}`);
      },

      async customerGetOrderRecommendations() {
        const endpoint = this.customerRecommendationTopOnly 
          ? `/orders/${this.customerRecommendationOrderId}/recommend_suppliers/?top_only=true`
          : `/orders/${this.customerRecommendationOrderId}/recommend_suppliers/`;
        
        this.customerOrderRecommendationsResponse = await this.makeRequest('GET', endpoint);
      },

      async customerGetGeneralRecommendations() {
        const endpoint = this.customerGeneralRecommendationData.top_only 
          ? '/recomendation?top_only=true'
          : '/recomendation';
        
        this.customerGeneralRecommendationsResponse = await this.makeRequest(
          'POST',
          endpoint,
          {
            okpd2: this.customerGeneralRecommendationData.okpd2,
            region: this.customerGeneralRecommendationData.region,
            law_type: this.customerGeneralRecommendationData.law_type
          }
        );
      },

      // make request
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
