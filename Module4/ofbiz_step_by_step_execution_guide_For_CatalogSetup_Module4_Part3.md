# OFBiz Catalog Setup — Complete Step-by-Step UI Execution Guide

This guide details the exact steps, navigation paths, and inputs required to execute the Catalog Setup assignment in OFBiz. You can use this as a reference whenever you forget how to navigate the menus.

---

## 🟢 ACTIVITY 1: Create Shoes Catalog

### 1. Create the Main Catalog
1. **Navigate to**: Main top menu ➔ **Catalog** ➔ **Catalogs** tab.
2. **Action**: Click the **Create New Catalog** button.
3. **Form Inputs**:
   - **Catalog Name**: `Shoes Catalog`
4. **Action**: Click the **Update** button.

### 2. Create the Categories
1. **Navigate to**: Main top menu ➔ **Catalog** ➔ **Categories** tab.
2. **Action**: Click the **Create New Category** button.
3. **Form Inputs**:
   - **Product Category Type**: Select `Catalog Category`
   - **Product Category ID**: Type a clear ID (e.g., `SHOES_ROOT`, `SHOES_MEN`, `M_SPORTS`, `SHOES_PROMO`)
   - **Product Category Name**: Type the name (e.g., `Browse Root`, `Men`, `MSports`, `Promotion`)
4. **Action**: Click the **Update** button.
*(Repeat this for all 11 categories: Browse Root, Men, MSports, MFormal, MParty, Women, WSports, WFormal, WParty, Children, Promotion).*

### 3. Link "Browse Root" to the Catalog
1. **Navigate to**: **Catalog** ➔ **Catalogs** tab.
2. **Search**: Find your "Shoes Catalog" and click its blue ID number.
3. **Navigate to**: Click the **Categories** sub-tab (in the row below the catalog name).
4. **Form Inputs (Add Product Category to Catalog)**:
   - **Product Category ID**: Type `SHOES_ROOT` (or use the list icon to look it up).
   - **ProdCatalog Category Type**: Select `Browse Root`.
5. **Action**: Click the **Add** button.

### 4. Create Category Hierarchy (Rollups)
1. **Navigate to**: **Catalog** ➔ **Categories** tab.
2. **Search**: Find the parent category (e.g., `SHOES_ROOT`) and click its blue ID number.
3. **Navigate to**: Click the **Rollup** sub-tab.
4. **Form Inputs (Category Rollup : Child Categories)**:
   - **Product Category ID**: Type the child ID (e.g., `SHOES_MEN`).
   - **From Date**: Click the calendar icon to automatically fill in the current date/time.
5. **Action**: Click the **Add** button.
*(Repeat this to link Men, Women, Children, and Promotion to Browse Root. Then link MSports, MFormal, MParty to Men, etc.)*

---

## 🟢 ACTIVITY 2: Define Features and Products

### 1. Create Feature Categories (Color and Size)
1. **Navigate to**: Main top menu ➔ **Catalog** ➔ **Features** tab ➔ **Feature Category** sub-tab.
2. **Action**: Click the light blue **New Feature Category** button.
3. **Form Inputs**:
   - **Description**: Type `Shoe Color`
   - **Parent Category**: (Leave blank)
4. **Action**: Click the **Submit** button.
*(Click New Feature Category again and repeat for `Shoe Size`).*

### 2. Create the Individual Features (Red, Black, Size 7, etc.)
1. **Navigate to**: **Catalog** ➔ **Features** tab ➔ **Feature Category** sub-tab.
2. **Search**: Type `Shoe` in the Description search box and click **Find**.
3. **Action**: Click the blue ID number next to "Shoe Color" in the search results.
4. **Form Inputs (Edit Feature)**:
   - **Feature Type**: Select `Color` (or `Size` if doing sizes).
   - **Description**: Type the value (e.g., `Red`, `Black`, `7`, `8`).
5. **Action**: Click the **Update** button.
*(Repeat until all colors and sizes are created).*

### 3. Create a Virtual Product (e.g., MS-1)
1. **Navigate to**: **Catalog** ➔ **Categories** tab.
2. **Search**: Find the target category (e.g., `M_SPORTS`) and click its ID.
3. **Navigate to**: Click the **Products** sub-tab.
4. **Action**: Click the light blue **Create Product in Category** link.
5. **Form Inputs**:
   - **Internal Name**: `MS-1`
   - **Product Name**: `Men Sports Shoes 1`
6. **Action**: Click the **Check Existing** button.
7. **Form Inputs (Next Screen)**:
   - **New Product ID**: Type exactly `MS-1`.
8. **Action**: Click the **Create New Product** button.
   *(If you get a "Duplicate Key" error, it means MS-1 already exists! Go to the left sidebar, click `MS-1` under Search Products, then go back to the Category's Products tab and use "Add Product Category Member" to link it).*
9. **Form Inputs (Product Edit Screen)**:
   - Expand the **Virtual Product** section by clicking the `(+)` icon.
   - **Is Virtual**: Change to `Y` (Yes).
   - **Is Variant**: Verify it is `N` (No).
10. **Action**: Scroll to the bottom and click **Update**.

### 4. Link Selectable Features to the Virtual Product
1. **Navigate to**: Inside the MS-1 product page, click the **Features** sub-tab (in the top row of tabs).
2. **Form Inputs (Add Product Features with ID section)**:
   - **ID**: Click the list icon (Lookup), search for your feature (e.g., `White`), and select it.
   - **Product Application Type**: Ensure this is set to `Selectable`.
3. **Action**: Click the teal **Add** button.
*(Repeat for all required colors and sizes for this specific shoe, e.g., White, Brown, 7, 8, 9, 10).*

### 5. Generate the Variant Products
1. **Navigate to**: Inside the MS-1 product page, click the **Variants** sub-tab (in the second row of tabs).
2. **Form Inputs (Quick Add Variants)**:
   - In the table, look at the **ALL** column on the far right.
   - **Check the boxes** for the specific Color/Size combinations needed for this shoe (e.g., Brown-7, Brown-8, White-7, etc.).
   - Leave the **New Product ID** fields completely blank (OFBiz will auto-generate them).
3. **Action**: Click the teal **Create** button at the bottom of the table.

*(Repeat steps 3, 4, and 5 for the remaining virtual products).*

---

## 🟢 ACTIVITY 3: Create Promotion

### 1. Add the Shoes to the "Promotion" Category
1. **Navigate to**: **Catalog** ➔ **Categories** tab.
2. **Search**: Find your **Promotion** category (e.g., `SHOES_PROMO`) and click its ID.
3. **Navigate to**: Click the **Products** sub-tab.
4. **Form Inputs (Add Product Category Member)**:
   - **Product ID**: Type the virtual product ID (e.g., `MS-1`).
   - **From Date**: Click the calendar icon to set the current date/time.
5. **Action**: Click the teal **Add** button.
*(Repeat this for MF-2, MP-1, WS-1, WF-1, WP-1, C-1, and C-2).*

### 2. Create the Promotion Rule (Promos Tab)
1. **Navigate to**: Main top menu ➔ **Catalog** ➔ **Promos** tab.
2. **Action**: Click the **Create New Product Promotion** button.
3. **Form Inputs**:
   - **Product Promo ID**: `10000` (or let it auto-generate)
   - **Promo Name**: `Shoes Special Discount`
   - **Promo Text**: `Special Offer on Selected Shoes!`
4. **Action**: Click **Update** (or Create).

### 3. Create the Rule Container & Link Category
1. **Navigate to**: Inside your new promotion, click the **Rules** tab (in the sub-menu).
2. **Form Inputs (Add New Promo Rule)**:
   - **Name**: `10% Discount`
3. **Action**: Click the teal **Add** button.
4. **Form Inputs (Promotion Categories - near the bottom)**:
   - **Product Category ID**: Click the lookup icon and select your **Promotion** category (`SHOES_PROMO`).
   - Ensure the dropdown says `Include`.
5. **Action**: Click the teal **Add** button next to it.

### 4. Configure Condition and Action
1. **Form Inputs (Conditions For Rule 00001)**:
   - **New** dropdown: Select `X quantity of Product` (or `Product quantity`).
   - **Is** dropdown: Select `>=` (greater than or equal to).
   - **Empty input box on the right**: Type `0` (meaning quantity > 0).
2. **Action**: Click the teal **Update** button next to the input box.
3. **Form Inputs (Actions For Rule 00001)**:
   - **New** dropdown: Select `Product discount percentage`.
   - **Amount**: Type `10` (for 10%).
4. **Action**: Click the teal **Create Action** button (or Update).

### 5. Link Promotion to Product Store
1. **Navigate to**: Inside your promotion, click the **Stores** tab (in the sub-menu).
2. **Form Inputs (Add Promotion to Store)**:
   - **Product Store**: Select your store (e.g., `9000`).
3. **Action**: Click the teal **Add** button.

---

## 🟢 ACTIVITY 5: XML Data Load (Alternative to UI)
If you ever want to load everything at once instead of clicking through the UI:
1. **Navigate to**: Main Application Menu (top right) ➔ **Webtools**.
2. **Navigate to**: Click the **Import/Export** tab ➔ **XML Data Import**.
3. **Form Inputs**:
   - **Absolute File path**: Type the exact path to your XML file (e.g., `/home/rachitaarsh/WorkSpace/Sand-box/ofbiz_catalog_activities_1_to_3.xml`).
4. **Action**: Click **Import Data**.
