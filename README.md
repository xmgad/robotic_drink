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

   Create Markdown

2. **Create Markdown**
   - Save as margaritaDescription.md in descriptions/
   - Include sections: Ingredients and Description

3. **Add Image**
   - Save as margaritaDescription.png in images/

4. **Generate QR Code**
   - Link: ?cocktail=margarita
   - Save QR in qr-codes/

5. **Setup Payment/Stock**
   - Add new ingredients to DB if needed
   - Ensure payment system recognizes new item

## 6. Frontend

A minimalistic UI with:

1. Drink Image: Quick visual representation for the user.

2. Drink Name: Clearly identifies the selected drink (“Margarita,” “Zombie,” etc.).

3. Description & Ingredients: Pulled from the Markdown file, providing flavor notes, allergy info, or background.

4. “Order Now” Button:

   - Triggers the system to check ingredient availability.

   - Processes dummy payment if stock is available.

   - Queues the order for robotic preparation.

   - Disables the button or shows a success/error message.

If the order is placed successfully, the user sees an Order ID. If not, an error is displayed. Status changes (like “Awaiting Restock” or “Drink in Preparation”) can be displayed to the user as well.

## 7. Backend

Our Node.js Express backend coordinates data flow:

1. ### Dependencies

   - Express for HTTP routes.

   - sqlite3 for persistent data storage.

   - axios for HTTP requests (if needed).

   - EventEmitter (optional) for event-driven notifications.
  
2. ### Database

   #### orders table

| Field     | Type     | Description                                                                 |
|-----------|----------|-----------------------------------------------------------------------------|
| id        | INTEGER  | Unique auto-incrementing order ID                                           |
| details   | TEXT     | JSON or text details describing the drink (e.g., cocktail name, extra info) |
| status    | TEXT     | Current status of the order (e.g., `queued`, `paid`, `robot_processing`, `finished`) |
| timestamp | DATETIME | Timestamp of when the order was created   

#### stock Table (Optional)

| Field        | Type    | Description                                  |
|--------------|---------|----------------------------------------------|
| ingredientId | INTEGER | Unique identifier for each ingredient        |
| quantity     | INTEGER | How many units are currently in stock        |

#### callbacks Table (Optional for CPEE)

| Field   | Type    | Description                                                      |
|---------|---------|------------------------------------------------------------------|
| id      | INTEGER | Unique auto-incrementing callback ID                             |
| address | TEXT    | Callback URL or endpoint for future notifications (e.g., CPEE)   |


3. ### API Routes

- **POST** `/order`  
  Creates a new cocktail order.

- **PUT** `/check-stock/:orderId`  
  Verifies ingredient availability.

- **PUT** `/restock-request/:ingredientId`  
  Requests restock for a missing ingredient.

- **PUT** `/payment/:orderId`  
  Mock payment endpoint.

- **GET** `/work-order`  
  Robot (or CPEE) fetches the oldest open order.

- **PUT** `/finished/:orderId`  
  Marks an order as finished.

4. ### Event Handling

- **On a new order**: Emit an `orderAvailable` event (optional).  
- **On restock**: Trigger a re-check of waiting orders.  
- **On payment success**: Update the order’s status to `paid`.

## 8. CPEE (Cloud Process Execution Engine)

Our CPEE workflow coordinates each step:

1. **Check Stock** → If not available, restock.  
2. **Payment** → If it fails, notify the user; if successful, proceed.  
3. **Robot Flow** → `GET /work-order` + preparation.  
4. **Finish** → `PUT /finished/:orderId`.

The `cpee-graph.xml` file encodes this flow, covering:

- **Stock checks**  
- **Payment branches** (success/fail)  
- **Automated drink preparation**  
- **Looping** to find the next open order

## Conclusion
This README outlines a QR code-based robotic drink ordering system. By combining a simple frontend interface, a Node.js backend, and a workflow engine (e.g., CPEE), you can build a fully automated solution that checks stock, manages payments, triggers restocking, and prepares drinks via a robot—while providing real-time user feedback.

Feel free to adapt the naming scheme, database structure, or logic as needed. Happy building!

