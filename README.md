# Restaurant Management System (RMS) - Frontend

A modern, responsive frontend for the Restaurant Management System (RMS) built with React.js, React Router, Tailwind CSS, and Nucleo Sharp Icons. This frontend provides a seamless user experience for managing restaurant operations, including orders, menu items, inventory, reservations, billing, and customer interactions.

---

## Features

### Admin & Staff Features
- Dashboard: Overview of key metrics (orders, sales, inventory alerts).
- Orders Management: View, update, and track orders (dine-in, takeaway, delivery).
- Menu Management: Add, edit, or delete menu categories and items.
- Inventory Management: Track stock levels, suppliers, and low-stock alerts.
- Billing & Invoicing: Generate and manage invoices for customers.
- Customer Management: View and manage customer details.
- Staff Management: Add, edit, or remove staff members.
- Reports: Sales, inventory, and customer reports with filtering options.
- Reservations: Manage table reservations with status updates.
- Settings: Configure system settings (admin-only).

### Customer Features
- Customer Dashboard: View personal order history and reservations.
- Online Ordering: Browse the menu and place orders.
- Reservations: Book a table for dine-in.
- Contact Admin: Send messages to the restaurant admin.

### UI/UX Features
- Dark/Light Mode: Toggle between themes.
- Responsive Design: Works on desktop, tablet, and mobile.
- Real-Time Notifications: Alerts for new orders, low inventory, and payments.
- Search & Filtering: Quickly find orders, customers, or menu items.
- Role-Based Access: Different views for admin, manager, staff, and customers.

---

## Tech Stack

- Framework: React.js (v18+)
- Routing: React Router DOM
- Styling: Tailwind CSS
- Icons: Nucleo Sharp
- State: React Context API
- Build Tool: Vite (recommended)

---

## Project Structure

rms-frontend/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── assets/
│       └── images/
├── src/
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Sidebar.jsx
│   │   │   ├── Header.jsx
│   │   │   └── Layout.jsx
│   │   ├── common/
│   │   │   ├── StatCard.jsx
│   │   │   ├── DataTable.jsx
│   │   │   ├── Modal.jsx
│   │   │   └── Button.jsx
│   │   ├── dashboard/
│   │   │   └── Dashboard.jsx
│   │   ├── orders/
│   │   │   ├── OrderList.jsx
│   │   │   ├── OrderForm.jsx
│   │   │   └── OrderDetails.jsx
│   │   ├── menu/
│   │   │   ├── MenuList.jsx
│   │   │   ├── MenuForm.jsx
│   │   │   └── MenuCategory.jsx
│   │   ├── inventory/
│   │   │   ├── InventoryList.jsx
│   │   │   └── InventoryForm.jsx
│   │   ├── customers/
│   │   │   ├── CustomerList.jsx
│   │   │   └── CustomerForm.jsx
│   │   ├── billing/
│   │   │   ├── InvoiceList.jsx
│   │   │   └── InvoiceForm.jsx
│   │   ├── reservations/
│   │   │   ├── ReservationList.jsx
│   │   │   └── ReservationForm.jsx
│   │   ├── reports/
│   │   │   ├── SalesReport.jsx
│   │   │   ├── InventoryReport.jsx
│   │   │   └── CustomerReport.jsx
│   │   ├── staff/
│   │   │   └── StaffList.jsx
│   │   ├── settings/
│   │   │   └── Settings.jsx
│   │   ├── customer/
│   │   │   ├── CustomerDashboard.jsx
│   │   │   └── ContactAdmin.jsx
│   │   └── auth/
│   │       ├── Login.jsx
│   │       └── Register.jsx
│   ├── context/
│   │   ├── ThemeContext.jsx
│   │   └── AuthContext.jsx
│   ├── hooks/
│   │   └── useFetch.js
│   ├── utils/
│   │   ├── api.js
│   │   └── constants.js
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── package.json
├── tailwind.config.js
├── vite.config.js
└── README.md

---

## Installation

### Prerequisites
- Node.js (v18+)
- npm or yarn
- Git

### Clone the Repository
```bash
git clone https://github.com/Shukal-01/Restaurant-Management-System-RMS-.git
cd Restaurant-Management-System-RMS-
```

### Install Dependencies
```bash
npm install
# or
yarn install
```

### Configure Environment Variables
Create a .env file in the root directory and add the following:

```env
# Backend API URL (update if backend is hosted elsewhere)
VITE_API_BASE_URL=http://localhost:5000/api/v1

# App title
VITE_APP_TITLE=Restaurant Management System
```

### Run the Development Server
```bash
npm run dev
# or
yarn dev
```

The app will start at http://localhost:5173.

---

## API Integration

The frontend connects to the RMS Backend. Ensure the backend is running and accessible at the URL specified in .env.

### Example API Calls
- Login: POST /auth/login
- Get Orders: GET /orders
- Create Menu Item: POST /menu/items
- Get Reports: GET /reports/sales

---

## UI Components

### Reusable Components
- StatCard: Displays metrics with icons and trends.
- DataTable: Searchable, paginated tables with actions.
- Modal: Reusable modal dialogs.
- Button: Styled buttons with variants.

### Layout
- Sidebar: Collapsible navigation with role-based access.
- Header: Theme toggle, notifications, and user profile.

---

## Available Scripts

- npm run dev: Start the development server.
- npm run build: Build for production.
- npm run preview: Preview the production build.
- npm run lint: Run ESLint for code quality checks.

---

## Dependencies

### Core Dependencies
- react & react-dom
- react-router-dom
- tailwindcss
- @tailwindcss/forms

### Icons
- nucleo-sharp

### Development Dependencies
- vite
- @vitejs/plugin-react
- autoprefixer & postcss

---

## Authentication

The app uses JWT-based authentication with the following roles:
- Admin: Full access to all features.
- Manager: Access to orders, menu, inventory, reports, and staff.
- Staff: Access to orders, customers, and reservations.
- Customer: Access to the customer dashboard and contact admin.

---

## Theme Toggle

The app supports dark and light modes with the following features:
- Toggle button in the header.
- Preference saved in localStorage.
- Smooth transitions between themes.

---

## Dashboard

The dashboard provides an overview of:
- Total Orders: Number of orders for the day/week/month.
- Revenue: Total sales.
- Pending Orders: Orders awaiting preparation.
- Low Stock Items: Inventory items below minimum stock level.
- Recent Activity: Notifications for orders, inventory, and billing.

---

## Responsive Design

The app is fully responsive and adapts to different screen sizes:
- Desktop: Full sidebar and header.
- Tablet: Collapsible sidebar.
- Mobile: Mobile-friendly navigation and touch targets.

---

## Deployment

### Build for Production
npm run build

This creates a dist folder with optimized files.

### Deploy to Netlify/Vercel
1. Push your code to GitHub.
2. Connect your repository to Netlify or Vercel.
3. Set the environment variables:
   - VITE_API_BASE_URL: Your backend API URL.
4. Deploy!

---

## Customization

- Modify colors in tailwind.config.js.
- Update the theme in src/index.css.
- Replace nucleo-sharp with another icon library if needed.

---

## Troubleshooting

### API Connection Errors
- Ensure the backend is running.
- Check VITE_API_BASE_URL in .env.
- Verify CORS settings in the backend.

### Build Failures
- Delete node_modules and reinstall dependencies.
- Check for syntax errors in your code.

### Styling Issues
- Ensure Tailwind CSS is properly configured.
- Restart the Vite server after making changes to tailwind.config.js.

---

## License

This project is open-source and available under the MIT License.

---

## Contact

- Email: shukal01@example.com
- GitHub: [Shukal-01](https://github.com/Shukal-01)