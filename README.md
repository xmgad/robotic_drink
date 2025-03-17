# Robot-Driven Cocktail Ordering System (Praktikum W23)

## 1. Motivation

In this project, we extend the idea of QR code-based cocktail ordering with an **automated robotic preparation** system. Users can still scan stylish QR codes to view and order drinks; however, there are several new steps:

1. **Ingredient Availability Check**: Before a customer pays, the system verifies that enough ingredients are in stock.  
2. **Restocking**: If an ingredient is unavailable, the system requests a restock, allowing staff or an inventory system to replenish it.  
3. **Payment Flow**: A simple payment step (using a mock Stripe integration) ensures that the order is paid for before preparation.  
4. **Robot Preparation**: Once payment is confirmed and ingredients are available, a robot prepares the drink.  
5. **Status Notifications**: Customers are notified at key stages (e.g., Payment Received, Drink in Preparation, Ready for Pickup).

This approach not only demonstrates a fun, automated solution for serving cocktails but also shows how a workflow engine (like **CPEE**) can orchestrate each step — from initial order to restock handling to final drink completion.

---

## 2. How to Run

Below are the high-level steps to run and test the system locally or on a server:

1. **Deploy Files**  
   Place the `descriptions`, `images`, `qr-codes`, and `src` folders on your server (for instance, `lehre.bpm.in.tum.de` or `localhost`).

2. **Install & Start the Backend**  
   - Navigate to `./src/server`.  
   - Run `npm install` (or `npm i`) to install dependencies.  
   - Start the server with `node server.js`.  
   - By default, it listens on a configured port (e.g., `3000`), which you can adjust if needed.

3. **Access Frontend**  
   - Scan a QR code for a specific drink, or open the relevant page in your browser (e.g., `https://<your-server>/index.html?cocktail=margarita`).  
   - You’ll see the drink description, an image, and an **“Order Now”** button.

4. **Order Flow**  
   - Click **“Order Now”**.  
   - The system checks ingredient availability, processes payment if stock is good, and then queues the order for the robot.  
   - You’ll receive a success or error notification. If success, you get an order ID, and the button may be disabled until the next order.

5. **End-to-End Demo**  
   - In a real setting, the **robot** or a separate automated system fetches the open order (via the same backend API), prepares the drink, and marks the order finished.  
   - If an ingredient is missing, a **restock** flow is triggered.  
   - Once restocked, the system can proceed with payment and preparation.

---

## 3. Folders and Files

### 3.1. `descriptions`
Contains **Markdown files** describing each drink. Files follow the naming scheme `<urlSearchParameter>Description.md`, detailing:

- **Ingredients**  
- **Extended Description** (e.g., background, taste, origin)

### 3.2. `images`
Contains images of each drink. Supported image types include: `webp, jpg, jpeg, png, gif, svg`.

### 3.3. `qr-codes`
Holds the **QR codes** linking to each drink’s dedicated ordering page. These can be generated using online tools (e.g., **QRCode-Monkey**).

### 3.4. `src`
Contains the main **frontend** and **backend** code:

- `src/client/` – Minimal HTML, CSS, and JS for the UI.  
- `src/server/` – Node.js (Express) backend, including database logic and routes.

### 3.5. `cpee-graph.xml`
Contains the **CPEE process definition** that can be imported into the **Cloud Process Execution Engine**. It defines the workflow for fetching orders, checking stock, processing payment, preparing drinks, and finishing orders.

---

## 4. Naming Scheme

Asset Type | Format | Example
--- | --- | --- | --- |--- |--- |--- |--- |--- |--- |--- |---
Description | <urlSearchParam>Description.md | margaritaDescription.md
--- | --- | --- | --- |--- |--- |--- |--- |--- |--- |---
Image | <urlSearchParam>Description.<ext>	 | margaritaDescription.png 

To keep the project consistent:

1. **Descriptions**:  
   `\`${urlSearchParameter}Description.md\``  
   *Example:* `margaritaDescription.md`

2. **Images**:  
   `\`${urlSearchParameter}Description.{format}\``  
   *Example:* `margaritaDescription.png`  
   *Formats:* `png, jpg, jpeg, webp, svg, gif`

3. **URL Search Parameter**  
   - Defined in `./src/script.js` in the `cocktails` map.  
   - Must match the name used in your descriptions, images, and any references in code.

---

## 5. Adding a Drink

1. **Add a Cocktail Reference**  
   In `./src/script.js`, update the `cocktails` variable with a new entry:
   ```js
   const cocktails = {
     ...
     margarita: 'Margarita'
   };



