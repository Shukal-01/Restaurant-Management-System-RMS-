---
name: "rms-system"
title: "Restaurant Management System - Full Stack"
type: "react"
---

import React, { useState, useEffect, useContext } from 'react';
import { BrowserRouter as Router, Routes, Route, Navigate, useLocation, useNavigate, Link } from 'react-router-dom';
import { 
  IconMenu as Menu, IconXmark as X, IconSunCloud as Sun, IconMoon as Moon, IconBell as Bell, IconUser as User, IconCircleLogout as LogOut, 
  IconHouseDashboard as Dashboard, IconCartShopping as ShoppingCart, IconBox2 as Box, IconUsers2 as Users, IconReceipt as Receipt, 
  IconChart as BarChart, IconHouseSettings as Settings, IconSpoonKnife as Restaurant, IconBoxArchive as Inventory, 
  IconPlus as Plus, IconSearchArea as Search, IconFilter as Filter, IconDotsVertical as MoreVertical, IconEditSquare as Edit, IconTrash2 as Trash2,
  IconEye as Eye, IconChefHat as ChefHat, IconCreditCard as CreditCard, IconCalendar as Calendar, IconFile as FileText,
  IconFileDownload as Printer, IconDownload as Download, IconFolderRefresh as RefreshCw, IconCircleCheck as CheckCircle, 
  IconCircleXmark as XCircle, IconClock as Clock, IconArrowUp as TrendingUp, IconArrowDown as TrendingDown,
  IconBox2 as Package, IconTag as Tag, IconCurrencyDollar as DollarSign, IconPercentSign as Percent, IconTimer as Time, IconCalendarDate as DateRange
} from 'nucleo-sharp';

// ============================================
// THEME CONTEXT (Dark/Light Mode)
// ============================================
const ThemeContext = React.createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    const savedTheme = localStorage.getItem('rms-theme');
    return savedTheme || 'dark';
  });

  useEffect(() => {
    document.documentElement.classList.remove('light', 'dark');
    document.documentElement.classList.add(theme);
    localStorage.setItem('rms-theme', theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prev => prev === 'dark' ? 'light' : 'dark');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// ============================================
// AUTH CONTEXT
// ============================================
const AuthContext = React.createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(() => {
    const savedUser = localStorage.getItem('rms-user');
    return savedUser ? JSON.parse(savedUser) : null;
  });

  const login = async (email, password) => {
    const mockUser = {
      user_id: '1',
      first_name: 'Admin',
      last_name: 'User',
      email: 'admin@example.com',
      role: { name: 'admin' },
      token: 'mock-jwt-token'
    };
    
    setUser(mockUser);
    localStorage.setItem('rms-user', JSON.stringify(mockUser));
    return mockUser;
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('rms-user');
  };

  const isAuthenticated = !!user;
  const isAdmin = user?.role?.name === 'admin';
  const isManager = user?.role?.name === 'manager' || isAdmin;
  const isStaff = user?.role?.name === 'staff' || isManager;

  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated, isAdmin, isManager, isStaff }}>
      {children}
    </AuthContext.Provider>
  );
}

// ============================================
// SIDEBAR COMPONENT
// ============================================
function Sidebar() {
  const { user, logout } = useContext(AuthContext);
  const [isCollapsed, setIsCollapsed] = useState(false);
  const location = useLocation();

  const navItems = [
    { path: '/', icon: Dashboard, label: 'Dashboard', roles: ['admin', 'manager', 'staff'] },
    { path: '/orders', icon: ShoppingCart, label: 'Orders', roles: ['admin', 'manager', 'staff'] },
    { path: '/menu', icon: Restaurant, label: 'Menu', roles: ['admin', 'manager'] },
    { path: '/inventory', icon: Box, label: 'Inventory', roles: ['admin', 'manager'] },
    { path: '/billing', icon: Receipt, label: 'Billing', roles: ['admin', 'manager', 'staff'] },
    { path: '/customers', icon: Users, label: 'Customers', roles: ['admin', 'manager', 'staff'] },
    { path: '/staff', icon: ChefHat, label: 'Staff', roles: ['admin', 'manager'] },
    { path: '/reports', icon: BarChart, label: 'Reports', roles: ['admin', 'manager'] },
    { path: '/reservations', icon: Calendar, label: 'Reservations', roles: ['admin', 'manager', 'staff'] },
    { path: '/settings', icon: Settings, label: 'Settings', roles: ['admin'] },
  ];

  // Customer navigation items (shown for all users)
  const customerNavItems = [
    { path: '/customer-dashboard', icon: Restaurant, label: 'Customer Portal', roles: ['customer', 'admin', 'manager', 'staff'] },
    { path: '/contact-admin', icon: Bell, label: 'Contact Admin', roles: ['customer', 'admin', 'manager', 'staff'] },
  ];

  // Combine nav items
  const allNavItems = [...navItems];
  
  // Add customer items if user is not authenticated (public access)
  if (!user) {
    allNavItems.push(...customerNavItems);
  }

  const filteredNavItems = allNavItems.filter(item => 
    user?.role?.name ? item.roles.includes(user.role.name) : true
  );

  return (
    <aside className={`fixed left-0 top-0 h-full bg-gradient-to-b from-slate-900 to-slate-800 text-slate-200 transition-all duration-300 z-50 ${isCollapsed ? 'w-20' : 'w-64'}`}>
      <div className="p-4 border-b border-slate-700 flex items-center justify-between">
        {!isCollapsed && (
          <div className="flex items-center gap-2">
            <Restaurant className="w-6 h-6 text-blue-400" />
            <span className="font-bold text-lg">RMS</span>
          </div>
        )}
        <button 
          onClick={() => setIsCollapsed(!isCollapsed)}
          className="p-1 rounded hover:bg-slate-700"
        >
          {isCollapsed ? <Menu className="w-5 h-5" /> : <X className="w-5 h-5" />}
        </button>
      </div>

      <nav className="p-4">
        <ul className="space-y-2">
          {filteredNavItems.map((item) => {
            const isActive = location.pathname === item.path || location.pathname.startsWith(`${item.path}/`);
            const Icon = item.icon;
            
            return (
              <li key={item.path}>
                <Link 
                  to={item.path}
                  className={`flex items-center gap-3 px-3 py-2 rounded-lg transition-colors ${
                    isActive 
                      ? 'bg-blue-600 text-white' 
                      : 'hover:bg-slate-700 text-slate-300'
                  }`}
                >
                  <Icon className={`w-5 h-5 ${isActive ? 'text-white' : 'text-slate-400'}`} />
                  {!isCollapsed && <span>{item.label}</span>}
                </Link>
              </li>
            );
          })}
        </ul>
      </nav>

      <div className="absolute bottom-0 left-0 right-0 p-4 border-t border-slate-700 bg-slate-800">
        {!isCollapsed && (
          <div className="mb-4">
            <h4 className="text-xs font-medium text-slate-400 uppercase tracking-wider mb-2">Customer Portal</h4>
            <div className="space-y-1">
              <Link 
                to="/customer-dashboard"
                className="flex items-center gap-3 px-3 py-2 rounded-lg text-slate-300 hover:bg-slate-700 transition-colors text-sm"
              >
                <Restaurant className="w-4 h-4" />
                <span>Customer Dashboard</span>
              </Link>
              <Link 
                to="/contact-admin"
                className="flex items-center gap-3 px-3 py-2 rounded-lg text-slate-300 hover:bg-slate-700 transition-colors text-sm"
              >
                <Bell className="w-4 h-4" />
                <span>Contact Admin</span>
              </Link>
            </div>
          </div>
        )}
        
        <div className={`flex items-center gap-3 ${isCollapsed ? 'justify-center' : ''}`}>
          <User className="w-5 h-5 text-slate-400" />
          {!isCollapsed && (
            <div>
              <div className="font-medium text-sm">{user?.first_name} {user?.last_name}</div>
              <div className="text-xs text-slate-400 capitalize">{user?.role?.name}</div>
            </div>
          )}
          {!isCollapsed && (
            <button 
              onClick={logout}
              className="text-slate-400 hover:text-red-400 transition-colors ml-auto"
            >
              <LogOut className="w-5 h-5" />
            </button>
          )}
        </div>
      </div>
    </aside>
  );
}

// ============================================
// HEADER COMPONENT
// ============================================
function Header() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  const { user } = useContext(AuthContext);
  const [notifications, setNotifications] = useState([
    { id: 1, message: 'New order received from Table 5', time: '5 mins ago', type: 'order' },
    { id: 2, message: 'Low stock alert for Chicken', time: '1 hour ago', type: 'inventory' },
    { id: 3, message: 'Payment received for Invoice #INV-001', time: '2 hours ago', type: 'billing' }
  ]);
  const [showNotifications, setShowNotifications] = useState(false);

  return (
    <header className="fixed top-0 right-0 left-64 h-16 bg-white dark:bg-slate-800 border-b border-slate-200 dark:border-slate-700 z-40 transition-all duration-300">
      <div className="h-full px-6 flex items-center justify-between">
        <div className="flex items-center gap-4">
          <h1 className="text-xl font-bold text-slate-800 dark:text-slate-100">
            {document.title}
          </h1>
        </div>

        <div className="flex items-center gap-4">
          <button 
            onClick={toggleTheme}
            className="p-2 rounded-full hover:bg-slate-100 dark:hover:bg-slate-700 transition-colors"
          >
            {theme === 'dark' ? (
              <Sun className="w-5 h-5 text-yellow-500" />
            ) : (
              <Moon className="w-5 h-5 text-slate-600" />
            )}
          </button>

          <button 
            onClick={() => setShowNotifications(!showNotifications)}
            className="relative p-2 rounded-full hover:bg-slate-100 dark:hover:bg-slate-700 transition-colors"
          >
            <Bell className="w-5 h-5 text-slate-600 dark:text-slate-400" />
            {notifications.length > 0 && (
              <span className="absolute top-1 right-1 w-4 h-4 bg-red-500 text-white text-xs rounded-full flex items-center justify-center">
                {notifications.length}
              </span>
            )}
          </button>

          {showNotifications && (
            <div className="absolute top-16 right-20 w-80 bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-lg shadow-lg p-4">
              <div className="flex items-center justify-between mb-4">
                <h3 className="font-semibold text-slate-800 dark:text-slate-100">Notifications</h3>
                <button 
                  onClick={() => setNotifications([])}
                  className="text-sm text-blue-600 hover:text-blue-700"
                >
                  Mark all as read
                </button>
              </div>
              {notifications.length === 0 ? (
                <p className="text-slate-500 dark:text-slate-400 text-sm">No new notifications</p>
              ) : (
                <ul className="space-y-3 max-h-80 overflow-y-auto">
                  {notifications.map((n) => (
                    <li 
                      key={n.id} 
                      className="p-2 rounded hover:bg-slate-50 dark:hover:bg-slate-700 cursor-pointer"
                      onClick={() => setNotifications(notifications.filter(notification => notification.id !== n.id))}
                    >
                      <div className="flex items-start gap-3">
                        <div className={`w-2 h-2 rounded-full mt-1.5 flex-shrink-0 ${
                          n.type === 'order' ? 'bg-blue-500' :
                          n.type === 'inventory' ? 'bg-orange-500' :
                          n.type === 'billing' ? 'bg-green-500' : 'bg-slate-400'
                        }`}></div>
                        <div>
                          <p className="text-sm text-slate-700 dark:text-slate-300">{n.message}</p>
                          <p className="text-xs text-slate-400 mt-1">{n.time}</p>
                        </div>
                      </div>
                    </li>
                  ))}
                </ul>
              )}
            </div>
          )}

          <div className="flex items-center gap-2">
            <div className="w-8 h-8 bg-gradient-to-br from-blue-500 to-purple-600 rounded-full flex items-center justify-center">
              <span className="text-white font-medium text-sm">
                {user?.first_name?.charAt(0)}{user?.last_name?.charAt(0)}
              </span>
            </div>
          </div>
        </div>
      </div>
    </header>
  );
}

// ============================================
// LAYOUT COMPONENT
// ============================================
function Layout({ children }) {
  return (
    <div className="min-h-screen bg-slate-50 dark:bg-slate-900">
      <Sidebar />
      <Header />
      <main className="ml-64 pt-16 p-6">
        {children}
      </main>
    </div>
  );
}

// ============================================
// STAT CARD COMPONENT (Reusable)
// ============================================
function StatCard({ title, value, icon: Icon, color, trend, trendValue }) {
  return (
    <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
      <div className="flex items-center justify-between">
        <div>
          <p className="text-sm text-slate-500 dark:text-slate-400">{title}</p>
          <p className="text-2xl font-bold text-slate-800 dark:text-slate-100 mt-1">{value}</p>
        </div>
        <div className={`w-10 h-10 ${color} rounded-lg flex items-center justify-center`}>
          <Icon className="w-5 h-5 text-white" />
        </div>
      </div>
      {trend && (
        <p className={`text-sm mt-4 flex items-center gap-1 ${
          trend === 'up' ? 'text-green-600' : 'text-red-600'
        }`}>
          {trend === 'up' ? <TrendingUp className="w-4 h-4" /> : <TrendingDown className="w-4 h-4" />}
          {trendValue}
        </p>
      )}
    </div>
  );
}

// ============================================
// DATA TABLE COMPONENT (Reusable)
// ============================================
function DataTable({ columns, data, keyField, onView, onEdit, onDelete, actions = ['view', 'edit', 'delete'] }) {
  const [searchQuery, setSearchQuery] = useState('');
  const [currentPage, setCurrentPage] = useState(1);
  const itemsPerPage = 10;

  const filteredData = data.filter(row => {
    return Object.values(row).some(value => 
      String(value).toLowerCase().includes(searchQuery.toLowerCase())
    );
  });

  const totalPages = Math.ceil(filteredData.length / itemsPerPage);
  const paginatedData = filteredData.slice(
    (currentPage - 1) * itemsPerPage,
    currentPage * itemsPerPage
  );

  return (
    <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
      <div className="flex items-center justify-between mb-4">
        <div className="relative flex-1 max-w-md">
          <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-slate-400" />
          <input 
            type="text" 
            placeholder="Search..."
            value={searchQuery}
            onChange={(e) => { setSearchQuery(e.target.value); setCurrentPage(1); }}
            className="w-full pl-10 pr-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
        </div>
      </div>
      
      <div className="overflow-x-auto">
        <table className="w-full">
          <thead>
            <tr className="border-b border-slate-200 dark:border-slate-700">
              {columns.map(column => (
                <th 
                  key={column.key}
                  className="text-left py-3 px-4 text-sm font-medium text-slate-500 dark:text-slate-400"
                >
                  {column.label}
                </th>
              ))}
              {actions.length > 0 && (
                <th className="text-left py-3 px-4 text-sm font-medium text-slate-500 dark:text-slate-400">
                  Actions
                </th>
              )}
            </tr>
          </thead>
          <tbody>
            {paginatedData.length === 0 ? (
              <tr>
                <td colSpan={columns.length + (actions.length > 0 ? 1 : 0)} className="py-8 text-center text-slate-500 dark:text-slate-400">
                  No data found
                </td>
              </tr>
            ) : (
              paginatedData.map(row => (
                <tr 
                  key={row[keyField]}
                  className="border-b border-slate-100 dark:border-slate-700 hover:bg-slate-50 dark:hover:bg-slate-700"
                >
                  {columns.map(column => (
                    <td key={column.key} className="py-3 px-4 text-sm text-slate-800 dark:text-slate-100">
                      {column.render ? column.render(row[column.key], row) : row[column.key]}
                    </td>
                  ))}
                  {actions.length > 0 && (
                    <td className="py-3 px-4">
                      <div className="flex items-center gap-2">
                        {actions.includes('view') && onView && (
                          <button 
                            onClick={() => onView(row)}
                            className="p-1 rounded hover:bg-slate-100 dark:hover:bg-slate-700 text-slate-500"
                          >
                            <Eye className="w-4 h-4" />
                          </button>
                        )}
                        {actions.includes('edit') && onEdit && (
                          <button 
                            onClick={() => onEdit(row)}
                            className="p-1 rounded hover:bg-slate-100 dark:hover:bg-slate-700 text-slate-500"
                          >
                            <Edit className="w-4 h-4" />
                          </button>
                        )}
                        {actions.includes('delete') && onDelete && (
                          <button 
                            onClick={() => onDelete(row)}
                            className="p-1 rounded hover:bg-red-50 dark:hover:bg-red-900/20 text-red-500"
                          >
                            <Trash2 className="w-4 h-4" />
                          </button>
                        )}
                        {actions.includes('more') && (
                          <button className="p-1 rounded hover:bg-slate-100 dark:hover:bg-slate-700 text-slate-500">
                            <MoreVertical className="w-4 h-4" />
                          </button>
                        )}
                      </div>
                    </td>
                  )}
                </tr>
              ))
            )}
          </tbody>
        </table>
      </div>

      {totalPages > 1 && (
        <div className="mt-4 flex items-center justify-between">
          <p className="text-sm text-slate-500 dark:text-slate-400">
            Showing {((currentPage - 1) * itemsPerPage) + 1}-{Math.min(currentPage * itemsPerPage, filteredData.length)} of {filteredData.length}
          </p>
          <div className="flex items-center gap-2">
            <button 
              onClick={() => setCurrentPage(prev => Math.max(prev - 1, 1))}
              disabled={currentPage === 1}
              className="px-3 py-1 text-sm border border-slate-200 dark:border-slate-700 rounded hover:bg-slate-50 dark:hover:bg-slate-700 disabled:opacity-50"
            >
              Previous
            </button>
            <span className="px-3 py-1 text-sm text-slate-500 dark:text-slate-400">
              Page {currentPage} of {totalPages}
            </span>
            <button 
              onClick={() => setCurrentPage(prev => Math.min(prev + 1, totalPages))}
              disabled={currentPage === totalPages}
              className="px-3 py-1 text-sm border border-slate-200 dark:border-slate-700 rounded hover:bg-slate-50 dark:hover:bg-slate-700 disabled:opacity-50"
            >
              Next
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

// ============================================
// MODAL COMPONENT (Reusable)
// ============================================
function Modal({ title, isOpen, onClose, children, size = 'md' }) {
  if (!isOpen) return null;

  const sizeClasses = {
    sm: 'max-w-sm',
    md: 'max-w-md',
    lg: 'max-w-lg',
    xl: 'max-w-xl',
    full: 'max-w-4xl'
  };

  return (
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4">
      <div className={`bg-white dark:bg-slate-800 rounded-xl p-6 w-full ${sizeClasses[size]}`}>
        <div className="flex items-center justify-between mb-4">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100">{title}</h2>
          <button 
            onClick={onClose}
            className="p-1 rounded hover:bg-slate-100 dark:hover:bg-slate-700 text-slate-500"
          >
            <X className="w-5 h-5" />
          </button>
        </div>
        {children}
      </div>
    </div>
  );
}

// ============================================
// FORM INPUT COMPONENT (Reusable)
// ============================================
function FormInput({ label, type = 'text', value, onChange, placeholder, required = false, disabled = false, options = [] }) {
  if (type === 'select') {
    return (
      <div className="mb-4">
        <label className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">
          {label}{required && <span className="text-red-500">*</span>}
        </label>
        <select 
          value={value}
          onChange={onChange}
          disabled={disabled}
          className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          {placeholder && <option value="">{placeholder}</option>}
          {options.map(option => (
            <option key={option.value} value={option.value}>{option.label}</option>
          ))}
        </select>
      </div>
    );
  }

  if (type === 'textarea') {
    return (
      <div className="mb-4">
        <label className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">
          {label}{required && <span className="text-red-500">*</span>}
        </label>
        <textarea 
          value={value}
          onChange={onChange}
          placeholder={placeholder}
          disabled={disabled}
          rows={4}
          className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed"
        />
      </div>
    );
  }

  return (
    <div className="mb-4">
      <label className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">
        {label}{required && <span className="text-red-500">*</span>}
      </label>
      <input 
        type={type}
        value={value}
        onChange={onChange}
        placeholder={placeholder}
        disabled={disabled}
        required={required}
        className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed"
      />
    </div>
  );
}

// ============================================
// DASHBOARD PAGE
// ============================================
function DashboardPage() {
  const [stats, setStats] = useState({
    totalOrders: 128,
    totalRevenue: '₹1,28,000',
    totalCustomers: 45,
    lowStockItems: 8
  });

  const [recentOrders, setRecentOrders] = useState([
    { id: 1, orderNumber: 'ORD-20260726-001', customer: 'John Doe', amount: '₹2,450', status: 'Served', time: '10:30 AM' },
    { id: 2, orderNumber: 'ORD-20260726-002', customer: 'Priya Sharma', amount: '₹1,800', status: 'Preparing', time: '11:15 AM' },
    { id: 3, orderNumber: 'ORD-20260726-003', customer: 'Rahul Gupta', amount: '₹3,200', status: 'Pending', time: '12:00 PM' },
    { id: 4, orderNumber: 'ORD-20260726-004', customer: 'Sneha Patel', amount: '₹1,500', status: 'Served', time: '12:30 PM' },
    { id: 5, orderNumber: 'ORD-20260726-005', customer: 'Amit Kumar', amount: '₹2,800', status: 'Ready', time: '01:00 PM' },
  ]);

  const [salesData, setSalesData] = useState({
    labels: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'],
    values: [12, 19, 15, 25, 22, 30, 28]
  });

  const [orderStatus, setOrderStatus] = useState([
    { label: 'Served', value: 75, color: 'green' },
    { label: 'Preparing', value: 25, color: 'blue' },
    { label: 'Pending', value: 8, color: 'orange' },
    { label: 'Cancelled', value: 2, color: 'red' }
  ]);

  const orderColumns = [
    { key: 'orderNumber', label: 'Order #' },
    { key: 'customer', label: 'Customer' },
    { key: 'amount', label: 'Amount' },
    { 
      key: 'status', 
      label: 'Status',
      render: (value) => (
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
          value === 'Served' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
          value === 'Preparing' ? 'bg-blue-100 text-blue-700 dark:bg-blue-900 dark:text-blue-300' :
          value === 'Ready' ? 'bg-purple-100 text-purple-700 dark:bg-purple-900 dark:text-purple-300' :
          value === 'Pending' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
          'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300'
        }`}>
          {value}
        </span>
      )
    },
    { key: 'time', label: 'Time' }
  ];

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Dashboard</h1>
        <p className="text-slate-500 dark:text-slate-400 mt-1">Welcome back! Here's what's happening today.</p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <StatCard 
          title="Total Orders" 
          value={stats.totalOrders} 
          icon={ShoppingCart} 
          color="bg-blue-500" 
          trend="up" 
          trendValue="+12% from yesterday"
        />
        <StatCard 
          title="Total Revenue" 
          value={stats.totalRevenue} 
          icon={CreditCard} 
          color="bg-green-500" 
          trend="up" 
          trendValue="+8% from yesterday"
        />
        <StatCard 
          title="Total Customers" 
          value={stats.totalCustomers} 
          icon={Users} 
          color="bg-purple-500" 
          trend="up" 
          trendValue="+3 new customers"
        />
        <StatCard 
          title="Low Stock Items" 
          value={stats.lowStockItems} 
          icon={Box} 
          color="bg-orange-500" 
          trend="down" 
          trendValue="Needs attention"
        />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <div className="flex items-center justify-between mb-4">
            <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100">Weekly Sales</h2>
            <select className="px-3 py-1 text-sm border border-slate-200 dark:border-slate-700 rounded bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100">
              <option>This Week</option>
              <option>Last Week</option>
              <option>This Month</option>
            </select>
          </div>
          <div className="h-64 flex items-end gap-2">
            {salesData.labels.map((label, index) => (
              <div key={label} className="flex-1 flex flex-col items-center gap-2">
                <div 
                  className="w-full bg-blue-500 rounded-t" 
                  style={{ height: `${salesData.values[index] * 2}px` }}
                ></div>
                <span className="text-xs text-slate-500 dark:text-slate-400">{label}</span>
              </div>
            ))}
          </div>
        </div>

        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">Order Status</h2>
          <div className="space-y-3">
            {orderStatus.map((item) => (
              <div key={item.label} className="flex items-center gap-4">
                <div className={`w-3 h-3 rounded-full bg-${item.color}-500 flex-shrink-0`}></div>
                <span className="text-sm text-slate-600 dark:text-slate-300 flex-1">{item.label}</span>
                <div className="w-48 h-2 bg-slate-200 dark:bg-slate-700 rounded-full overflow-hidden">
                  <div className={`w-[${item.value}%] h-full bg-${item.color}-500 rounded-full`}></div>
                </div>
                <span className="text-sm text-slate-500 dark:text-slate-400 w-12 text-right">{item.value}%</span>
              </div>
            ))}
          </div>
        </div>
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <div className="flex items-center justify-between mb-4">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100">Recent Orders</h2>
          <button className="flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
            <Plus className="w-4 h-4" />
            <span>New Order</span>
          </button>
        </div>
        <DataTable 
          columns={orderColumns} 
          data={recentOrders} 
          keyField="id" 
          actions={['view', 'edit']}
        />
      </div>
    </div>
  );
}

// ============================================
// ORDERS PAGE
// ============================================
function OrdersPage() {
  const { isAdmin, isManager } = useContext(AuthContext);
  const [orders, setOrders] = useState([
    { 
      id: 1, orderNumber: 'ORD-20260726-001', table: 'Table 1', customer: 'John Doe', 
      items: 3, amount: '₹2,450', discount: '₹0', tax: '₹245', total: '₹2,695',
      status: 'Served', paymentStatus: 'Paid', time: '10:30 AM', staff: 'Priya Sharma'
    },
    { 
      id: 2, orderNumber: 'ORD-20260726-002', table: 'Table 2', customer: 'Priya Sharma', 
      items: 2, amount: '₹1,800', discount: '₹100', tax: '₹180', total: '₹1,880',
      status: 'Preparing', paymentStatus: 'Pending', time: '11:15 AM', staff: 'Rahul Gupta'
    },
    { 
      id: 3, orderNumber: 'ORD-20260726-003', table: 'Table 3', customer: 'Rahul Gupta', 
      items: 4, amount: '₹3,200', discount: '₹200', tax: '₹320', total: '₹3,320',
      status: 'Pending', paymentStatus: 'Unpaid', time: '12:00 PM', staff: 'John Doe'
    },
  ]);
  const [statusFilter, setStatusFilter] = useState('all');
  const [paymentFilter, setPaymentFilter] = useState('all');
  const [showOrderModal, setShowOrderModal] = useState(false);
  const [selectedOrder, setSelectedOrder] = useState(null);

  const statusOptions = ['All', 'Pending', 'Preparing', 'Ready', 'Served', 'Cancelled'];
  const paymentOptions = ['All', 'Unpaid', 'Partial', 'Paid', 'Refunded'];

  const filteredOrders = orders.filter(order => {
    const matchesStatus = statusFilter === 'all' || order.status.toLowerCase() === statusFilter.toLowerCase();
    const matchesPayment = paymentFilter === 'all' || order.paymentStatus.toLowerCase() === paymentFilter.toLowerCase();
    return matchesStatus && matchesPayment;
  });

  const orderColumns = [
    { key: 'orderNumber', label: 'Order #' },
    { key: 'table', label: 'Table' },
    { key: 'customer', label: 'Customer' },
    { key: 'items', label: 'Items' },
    { key: 'total', label: 'Total' },
    { 
      key: 'status', 
      label: 'Status',
      render: (value) => (
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
          value === 'Served' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
          value === 'Preparing' ? 'bg-blue-100 text-blue-700 dark:bg-blue-900 dark:text-blue-300' :
          value === 'Ready' ? 'bg-purple-100 text-purple-700 dark:bg-purple-900 dark:text-purple-300' :
          value === 'Pending' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
          'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300'
        }`}>
          {value}
        </span>
      )
    },
    { 
      key: 'paymentStatus', 
      label: 'Payment',
      render: (value) => (
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
          value === 'Paid' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
          value === 'Partial' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
          value === 'Unpaid' ? 'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300' :
          'bg-slate-100 text-slate-700 dark:bg-slate-900 dark:text-slate-300'
        }`}>
          {value}
        </span>
      )
    },
    { key: 'time', label: 'Time' },
    { key: 'staff', label: 'Staff' }
  ];

  const handleViewOrder = (order) => {
    setSelectedOrder(order);
    setShowOrderModal(true);
  };

  const handleEditOrder = (order) => {
    alert(`Edit order: ${order.orderNumber}`);
  };

  const handleStatusChange = (orderId, newStatus) => {
    setOrders(orders.map(order => 
      order.id === orderId ? { ...order, status: newStatus } : order
    ));
    setShowOrderModal(false);
  };

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Orders</h1>
        <p className="text-slate-500 dark:text-slate-400 mt-1">Manage all customer orders</p>
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
          <div className="relative">
            <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-slate-400" />
            <input 
              type="text" 
              placeholder="Search orders..."
              className="w-full pl-10 pr-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
          <select 
            value={statusFilter}
            onChange={(e) => setStatusFilter(e.target.value)}
            className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            {statusOptions.map(option => (
              <option key={option.toLowerCase()} value={option.toLowerCase()}>
                {option}
              </option>
            ))}
          </select>
          <select 
            value={paymentFilter}
            onChange={(e) => setPaymentFilter(e.target.value)}
            className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            {paymentOptions.map(option => (
              <option key={option.toLowerCase()} value={option.toLowerCase()}>
                {option}
              </option>
            ))}
          </select>
          {(isAdmin || isManager) && (
            <button className="flex items-center justify-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
              <Plus className="w-4 h-4" />
              <span>New Order</span>
            </button>
          )}
        </div>
      </div>

      <DataTable 
        columns={orderColumns} 
        data={filteredOrders} 
        keyField="id" 
        onView={handleViewOrder}
        onEdit={handleEditOrder}
        actions={['view', 'edit']}
      />

      <Modal 
        title={`Order #${selectedOrder?.orderNumber || ''}`} 
        isOpen={showOrderModal} 
        onClose={() => setShowOrderModal(false)}
        size="lg"
      >
        {selectedOrder && (
          <div className="space-y-4">
            <div className="grid grid-cols-1 md:grid-cols-3 gap-4 pb-4 border-b border-slate-200 dark:border-slate-700">
              <div>
                <p className="text-sm text-slate-500 dark:text-slate-400">Order Number</p>
                <p className="font-medium text-slate-800 dark:text-slate-100">{selectedOrder.orderNumber}</p>
              </div>
              <div>
                <p className="text-sm text-slate-500 dark:text-slate-400">Date & Time</p>
                <p className="font-medium text-slate-800 dark:text-slate-100">26 Jul 2026, {selectedOrder.time}</p>
              </div>
              <div>
                <p className="text-sm text-slate-500 dark:text-slate-400">Staff</p>
                <p className="font-medium text-slate-800 dark:text-slate-100">{selectedOrder.staff}</p>
              </div>
            </div>

            <div>
              <h3 className="font-semibold text-slate-800 dark:text-slate-100 mb-3">Order Items ({selectedOrder.items})</h3>
              <div className="space-y-3">
                {[1, 2, 3].slice(0, selectedOrder.items).map((item, index) => (
                  <div key={index} className="flex items-center justify-between p-3 bg-slate-50 dark:bg-slate-700 rounded-lg">
                    <div className="flex-1">
                      <p className="font-medium text-slate-800 dark:text-slate-100">Paneer Tikka Masala</p>
                      <p className="text-sm text-slate-500 dark:text-slate-400">Quantity: 2 | ₹450 each</p>
                    </div>
                    <p className="font-medium text-green-600">₹900</p>
                  </div>
                ))}
              </div>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 gap-4 pt-4 border-t border-slate-200 dark:border-slate-700">
              <div className="space-y-2">
                <div className="flex justify-between">
                  <p className="text-sm text-slate-500 dark:text-slate-400">Subtotal</p>
                  <p className="text-sm text-slate-800 dark:text-slate-100">{selectedOrder.amount}</p>
                </div>
                <div className="flex justify-between">
                  <p className="text-sm text-slate-500 dark:text-slate-400">Discount</p>
                  <p className="text-sm text-green-600">-{selectedOrder.discount}</p>
                </div>
                <div className="flex justify-between">
                  <p className="text-sm text-slate-500 dark:text-slate-400">Tax</p>
                  <p className="text-sm text-slate-800 dark:text-slate-100">{selectedOrder.tax}</p>
                </div>
              </div>
              <div className="space-y-2">
                <div className="flex justify-between">
                  <p className="text-sm text-slate-500 dark:text-slate-400">Total Amount</p>
                  <p className="font-semibold text-slate-800 dark:text-slate-100">{selectedOrder.total}</p>
                </div>
                <div className="flex justify-between">
                  <p className="text-sm text-slate-500 dark:text-slate-400">Payment Status</p>
                  <span className={`px-2 py-1 rounded-full text-xs font-medium ${
                    selectedOrder.paymentStatus === 'Paid' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
                    selectedOrder.paymentStatus === 'Partial' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
                    'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300'
                  }`}>
                    {selectedOrder.paymentStatus}
                  </span>
                </div>
                <div className="flex justify-between">
                  <p className="text-sm text-slate-500 dark:text-slate-400">Order Status</p>
                  <span className={`px-2 py-1 rounded-full text-xs font-medium ${
                    selectedOrder.status === 'Served' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
                    selectedOrder.status === 'Preparing' ? 'bg-blue-100 text-blue-700 dark:bg-blue-900 dark:text-blue-300' :
                    selectedOrder.status === 'Ready' ? 'bg-purple-100 text-purple-700 dark:bg-purple-900 dark:text-purple-300' :
                    selectedOrder.status === 'Pending' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
                    'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300'
                  }`}>
                    {selectedOrder.status}
                  </span>
                </div>
              </div>
            </div>

            {(isAdmin || isManager) && (
              <div className="flex items-center justify-end gap-2 pt-4 border-t border-slate-200 dark:border-slate-700">
                <button className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700">
                  Print Receipt
                </button>
                <select 
                  onChange={(e) => handleStatusChange(selectedOrder.id, e.target.value)}
                  className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100"
                  defaultValue={selectedOrder.status}
                >
                  <option value="Pending">Pending</option>
                  <option value="Preparing">Preparing</option>
                  <option value="Ready">Ready</option>
                  <option value="Served">Served</option>
                  <option value="Cancelled">Cancelled</option>
                </select>
                <button className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700">
                  Save Changes
                </button>
              </div>
            )}
          </div>
        )}
      </Modal>
    </div>
  );
}

// ============================================
// MENU PAGE
// ============================================
function MenuPage() {
  const { isAdmin, isManager } = useContext(AuthContext);
  const [categories, setCategories] = useState([
    { id: 1, name: 'Appetizers', description: 'Starters and snacks', itemCount: 8, isActive: true },
    { id: 2, name: 'Main Course', description: 'Primary dishes', itemCount: 12, isActive: true },
    { id: 3, name: 'Desserts', description: 'Sweet treats', itemCount: 5, isActive: true },
    { id: 4, name: 'Beverages', description: 'Drinks', itemCount: 10, isActive: true },
  ]);
  const [menuItems, setMenuItems] = useState([
    { 
      id: 1, name: 'Paneer Tikka', category: 'Appetizers', categoryId: 1,
      description: 'Grilled cottage cheese cubes with spices',
      price: 280, cost: 120, image: null,
      isVegetarian: true, isVegan: false, isGlutenFree: true,
      isAvailable: true, preparationTime: 15
    },
    { 
      id: 2, name: 'Chicken Biryani', category: 'Main Course', categoryId: 2,
      description: 'Fragrant basmati rice with chicken and spices',
      price: 350, cost: 180, image: null,
      isVegetarian: false, isVegan: false, isGlutenFree: true,
      isAvailable: true, preparationTime: 30
    },
    { 
      id: 3, name: 'Gulab Jamun', category: 'Desserts', categoryId: 3,
      description: 'Deep-fried milk-solid dumplings in sugar syrup',
      price: 80, cost: 30, image: null,
      isVegetarian: true, isVegan: false, isGlutenFree: false,
      isAvailable: true, preparationTime: 20
    },
  ]);
  const [selectedCategory, setSelectedCategory] = useState('all');
  const [showCategoryModal, setShowCategoryModal] = useState(false);
  const [showItemModal, setShowItemModal] = useState(false);
  const [editingCategory, setEditingCategory] = useState(null);
  const [editingItem, setEditingItem] = useState(null);

  const filteredItems = selectedCategory === 'all' 
    ? menuItems 
    : menuItems.filter(item => item.categoryId.toString() === selectedCategory);

  const handleCategorySubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const newCategory = {
      id: editingCategory?.id || categories.length + 1,
      name: formData.get('name'),
      description: formData.get('description'),
      itemCount: 0,
      isActive: formData.get('isActive') === 'on'
    };
    
    if (editingCategory) {
      setCategories(categories.map(c => c.id === newCategory.id ? newCategory : c));
    } else {
      setCategories([...categories, newCategory]);
    }
    
    setShowCategoryModal(false);
    setEditingCategory(null);
  };

  const handleItemSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const newItem = {
      id: editingItem?.id || menuItems.length + 1,
      name: formData.get('name'),
      category: categories.find(c => c.id.toString() === formData.get('categoryId'))?.name || '',
      categoryId: parseInt(formData.get('categoryId')) || 1,
      description: formData.get('description'),
      price: parseFloat(formData.get('price')) || 0,
      cost: parseFloat(formData.get('cost')) || 0,
      image: null,
      isVegetarian: formData.get('isVegetarian') === 'on',
      isVegan: formData.get('isVegan') === 'on',
      isGlutenFree: formData.get('isGlutenFree') === 'on',
      isAvailable: formData.get('isAvailable') === 'on',
      preparationTime: parseInt(formData.get('preparationTime')) || 0
    };
    
    if (editingItem) {
      setMenuItems(menuItems.map(item => item.id === newItem.id ? newItem : item));
    } else {
      setMenuItems([...menuItems, newItem]);
    }
    
    setShowItemModal(false);
    setEditingItem(null);
  };

  const handleDeleteCategory = (category) => {
    if (confirm(`Delete category "${category.name}"? This will not delete menu items.`)) {
      setCategories(categories.filter(c => c.id !== category.id));
    }
  };

  const handleDeleteItem = (item) => {
    if (confirm(`Delete menu item "${item.name}"?`)) {
      setMenuItems(menuItems.filter(i => i.id !== item.id));
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Menu Management</h1>
          <p className="text-slate-500 dark:text-slate-400 mt-1">Manage your restaurant menu</p>
        </div>
        {(isAdmin || isManager) && (
          <div className="flex items-center gap-2">
            <button 
              onClick={() => { setEditingCategory(null); setShowCategoryModal(true); }}
              className="flex items-center gap-2 px-4 py-2 bg-slate-100 dark:bg-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-200 dark:hover:bg-slate-600 transition-colors"
            >
              <Plus className="w-4 h-4" />
              <span>Add Category</span>
            </button>
            <button 
              onClick={() => { setEditingItem(null); setShowItemModal(true); }}
              className="flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
            >
              <Plus className="w-4 h-4" />
              <span>Add Menu Item</span>
            </button>
          </div>
        )}
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">Categories</h2>
        <div className="flex flex-wrap gap-2">
          <button 
            onClick={() => setSelectedCategory('all')}
            className={`px-4 py-2 rounded-lg text-sm font-medium transition-colors ${
              selectedCategory === 'all' 
                ? 'bg-blue-600 text-white' 
                : 'bg-slate-100 dark:bg-slate-700 text-slate-800 dark:text-slate-100 hover:bg-slate-200 dark:hover:bg-slate-600'
            }`}
          >
            All ({menuItems.length})
          </button>
          {categories.map(category => (
            <button 
              key={category.id}
              onClick={() => setSelectedCategory(category.id.toString())}
              className={`px-4 py-2 rounded-lg text-sm font-medium transition-colors ${
                selectedCategory === category.id.toString() 
                  ? 'bg-blue-600 text-white' 
                  : 'bg-slate-100 dark:bg-slate-700 text-slate-800 dark:text-slate-100 hover:bg-slate-200 dark:hover:bg-slate-600'
              }`}
            >
              {category.name} ({category.itemCount})
            </button>
          ))}
        </div>
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">
          Menu Items {selectedCategory !== 'all' && `in ${categories.find(c => c.id.toString() === selectedCategory)?.name}`}
        </h2>
        
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
          {filteredItems.map(item => (
            <div key={item.id} className="border border-slate-200 dark:border-slate-700 rounded-lg p-4 hover:shadow-md transition-shadow">
              <div className="flex items-start justify-between">
                <div className="flex-1">
                  <div className="flex items-center gap-2 mb-2">
                    <h3 className="font-semibold text-slate-800 dark:text-slate-100">{item.name}</h3>
                    {item.isVegetarian && (
                      <span className="px-2 py-0.5 bg-green-100 dark:bg-green-900 text-green-700 dark:text-green-300 text-xs rounded-full">
                        Veg
                      </span>
                    )}
                    {item.isVegan && (
                      <span className="px-2 py-0.5 bg-purple-100 dark:bg-purple-900 text-purple-700 dark:text-purple-300 text-xs rounded-full">
                        Vegan
                      </span>
                    )}
                    {item.isGlutenFree && (
                      <span className="px-2 py-0.5 bg-orange-100 dark:bg-orange-900 text-orange-700 dark:text-orange-300 text-xs rounded-full">
                        GF
                      </span>
                    )}
                  </div>
                  <p className="text-sm text-slate-500 dark:text-slate-400 mb-2">{item.description}</p>
                  <div className="flex items-center gap-4 text-sm text-slate-500 dark:text-slate-400">
                    <span>₹{item.price}</span>
                    <span>|</span>
                    <span>{item.preparationTime} mins</span>
                  </div>
                </div>
                <div className="flex items-center gap-1">
                  <span className={`px-2 py-0.5 text-xs rounded-full ${
                    item.isAvailable 
                      ? 'bg-green-100 dark:bg-green-900 text-green-700 dark:text-green-300' 
                      : 'bg-red-100 dark:bg-red-900 text-red-700 dark:text-red-300'
                  }`}>
                    {item.isAvailable ? 'Available' : 'Unavailable'}
                  </span>
                  {(isAdmin || isManager) && (
                    <>
                      <button 
                        onClick={() => { setEditingItem(item); setShowItemModal(true); }}
                        className="p-1 rounded hover:bg-slate-100 dark:hover:bg-slate-700"
                      >
                        <Edit className="w-4 h-4 text-slate-500" />
                      </button>
                      <button 
                        onClick={() => handleDeleteItem(item)}
                        className="p-1 rounded hover:bg-red-50 dark:hover:bg-red-900/20"
                      >
                        <Trash2 className="w-4 h-4 text-red-500" />
                      </button>
                    </>
                  )}
                </div>
              </div>
              <div className="mt-3 pt-3 border-t border-slate-100 dark:border-slate-700 flex items-center justify-between">
                <span className="text-sm text-slate-500 dark:text-slate-400">{item.category}</span>
                <span className="text-sm font-medium text-blue-600">Margin: ₹{item.price - item.cost}</span>
              </div>
            </div>
          ))}
        </div>
      </div>

      <Modal 
        title={editingCategory ? 'Edit Category' : 'Add Category'} 
        isOpen={showCategoryModal} 
        onClose={() => { setShowCategoryModal(false); setEditingCategory(null); }}
      >
        <form onSubmit={handleCategorySubmit} className="space-y-4">
          <FormInput 
            label="Category Name" 
            type="text" 
            value={editingCategory?.name || ''} 
            onChange={(e) => setEditingCategory({...editingCategory, name: e.target.value})}
            placeholder="e.g., Appetizers"
            required
          />
          <FormInput 
            label="Description" 
            type="textarea" 
            value={editingCategory?.description || ''} 
            onChange={(e) => setEditingCategory({...editingCategory, description: e.target.value})}
            placeholder="Brief description..."
          />
          <label className="flex items-center gap-2 cursor-pointer mb-4">
            <input 
              type="checkbox" 
              checked={editingCategory?.isActive || true}
              onChange={(e) => setEditingCategory({...editingCategory, isActive: e.target.checked})}
              className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
            />
            <span className="text-sm text-slate-700 dark:text-slate-300">Active</span>
          </label>
          <div className="flex items-center justify-end gap-2">
            <button 
              type="button"
              onClick={() => { setShowCategoryModal(false); setEditingCategory(null); }}
              className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
            >
              {editingCategory ? 'Update Category' : 'Add Category'}
            </button>
          </div>
        </form>
      </Modal>

      <Modal 
        title={editingItem ? 'Edit Menu Item' : 'Add Menu Item'} 
        isOpen={showItemModal} 
        onClose={() => { setShowItemModal(false); setEditingItem(null); }}
        size="lg"
      >
        <form onSubmit={handleItemSubmit} className="space-y-4">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <FormInput 
              label="Item Name" 
              type="text" 
              value={editingItem?.name || ''} 
              onChange={(e) => setEditingItem({...editingItem, name: e.target.value})}
              placeholder="e.g., Paneer Tikka"
              required
            />
            <FormInput 
              label="Category" 
              type="select" 
              value={editingItem?.categoryId || ''} 
              onChange={(e) => setEditingItem({...editingItem, categoryId: parseInt(e.target.value)})}
              placeholder="Select category"
              required
              options={categories.map(c => ({ value: c.id, label: c.name }))}
            />
          </div>
          
          <FormInput 
            label="Description" 
            type="textarea" 
            value={editingItem?.description || ''} 
            onChange={(e) => setEditingItem({...editingItem, description: e.target.value})}
            placeholder="Detailed description..."
          />

          <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
            <FormInput 
              label="Price (₹)" 
              type="number" 
              value={editingItem?.price || ''} 
              onChange={(e) => setEditingItem({...editingItem, price: parseFloat(e.target.value) || 0})}
              placeholder="280"
              required
            />
            <FormInput 
              label="Cost (₹)" 
              type="number" 
              value={editingItem?.cost || ''} 
              onChange={(e) => setEditingItem({...editingItem, cost: parseFloat(e.target.value) || 0})}
              placeholder="120"
            />
            <FormInput 
              label="Preparation Time (mins)" 
              type="number" 
              value={editingItem?.preparationTime || ''} 
              onChange={(e) => setEditingItem({...editingItem, preparationTime: parseInt(e.target.value) || 0})}
              placeholder="15"
            />
          </div>

          <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
            <label className="flex items-center gap-2 cursor-pointer">
              <input 
                type="checkbox" 
                checked={editingItem?.isVegetarian || false}
                onChange={(e) => setEditingItem({...editingItem, isVegetarian: e.target.checked})}
                className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
              />
              <span className="text-sm text-slate-700 dark:text-slate-300">Vegetarian</span>
            </label>
            <label className="flex items-center gap-2 cursor-pointer">
              <input 
                type="checkbox" 
                checked={editingItem?.isVegan || false}
                onChange={(e) => setEditingItem({...editingItem, isVegan: e.target.checked})}
                className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
              />
              <span className="text-sm text-slate-700 dark:text-slate-300">Vegan</span>
            </label>
            <label className="flex items-center gap-2 cursor-pointer">
              <input 
                type="checkbox" 
                checked={editingItem?.isGlutenFree || false}
                onChange={(e) => setEditingItem({...editingItem, isGlutenFree: e.target.checked})}
                className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
              />
              <span className="text-sm text-slate-700 dark:text-slate-300">Gluten Free</span>
            </label>
            <label className="flex items-center gap-2 cursor-pointer">
              <input 
                type="checkbox" 
                checked={editingItem?.isAvailable || true}
                onChange={(e) => setEditingItem({...editingItem, isAvailable: e.target.checked})}
                className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
              />
              <span className="text-sm text-slate-700 dark:text-slate-300">Available</span>
            </label>
          </div>

          <div className="flex items-center justify-end gap-2">
            <button 
              type="button"
              onClick={() => { setShowItemModal(false); setEditingItem(null); }}
              className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
            >
              {editingItem ? 'Update Menu Item' : 'Add Menu Item'}
            </button>
          </div>
        </form>
      </Modal>
    </div>
  );
}

// ============================================
// INVENTORY PAGE
// ============================================
function InventoryPage() {
  const { isAdmin, isManager } = useContext(AuthContext);
  const [inventory, setInventory] = useState([
    { 
      id: 1, name: 'Paneer', category: 'Dairy', unit: 'kg', 
      currentStock: 5.2, minStockLevel: 2, costPerUnit: 450,
      supplier: 'ABC Dairy', supplierPhone: '+91 9876543210',
      status: 'In Stock', lastUpdated: '2 days ago'
    },
    { 
      id: 2, name: 'Chicken', category: 'Meat', unit: 'kg', 
      currentStock: 3.5, minStockLevel: 5, costPerUnit: 300,
      supplier: 'XYZ Poultry', supplierPhone: '+91 9876543211',
      status: 'Low Stock', lastUpdated: '1 day ago'
    },
    { 
      id: 3, name: 'Basmati Rice', category: 'Grains', unit: 'kg', 
      currentStock: 20, minStockLevel: 10, costPerUnit: 80,
      supplier: 'PQR Foods', supplierPhone: '+91 9876543212',
      status: 'In Stock', lastUpdated: '3 days ago'
    },
  ]);
  const [statusFilter, setStatusFilter] = useState('all');
  const [showItemModal, setShowItemModal] = useState(false);
  const [showPurchaseModal, setShowPurchaseModal] = useState(false);
  const [editingItem, setEditingItem] = useState(null);

  const filteredInventory = inventory.filter(item => {
    const matchesStatus = statusFilter === 'all' || item.status.toLowerCase() === statusFilter.toLowerCase();
    return matchesStatus;
  });

  const inventoryColumns = [
    { key: 'name', label: 'Item' },
    { key: 'category', label: 'Category' },
    { 
      key: 'currentStock', 
      label: 'Current Stock',
      render: (value, row) => `${value} ${row.unit}`
    },
    { 
      key: 'minStockLevel', 
      label: 'Min Stock',
      render: (value, row) => `${value} ${row.unit}`
    },
    { 
      key: 'costPerUnit', 
      label: 'Cost/Unit',
      render: (value) => `₹${value}`
    },
    { key: 'supplier', label: 'Supplier' },
    { 
      key: 'status', 
      label: 'Status',
      render: (value) => (
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
          value === 'In Stock' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
          value === 'Low Stock' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
          'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300'
        }`}>
          {value}
        </span>
      )
    },
    { key: 'lastUpdated', label: 'Last Updated' }
  ];

  const handlePurchase = (item) => {
    setEditingItem(item);
    setShowPurchaseModal(true);
  };

  const handlePurchaseSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const quantity = parseFloat(formData.get('quantity')) || 0;
    const newStock = (editingItem?.currentStock || 0) + quantity;
    
    setInventory(inventory.map(item => 
      item.id === editingItem?.id 
        ? { 
            ...item, 
            currentStock: newStock,
            status: newStock >= item.minStockLevel ? 'In Stock' : newStock > 0 ? 'Low Stock' : 'Out of Stock',
            lastUpdated: 'Just now'
          }
        : item
    ));
    
    setShowPurchaseModal(false);
    setEditingItem(null);
    alert(`Purchase recorded! New stock: ${newStock} ${editingItem?.unit}`);
  };

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Inventory</h1>
        <p className="text-slate-500 dark:text-slate-400 mt-1">Track your stock levels and suppliers</p>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <StatCard 
          title="Total Items" 
          value={inventory.length} 
          icon={Package} 
          color="bg-blue-500"
        />
        <StatCard 
          title="Low Stock" 
          value={inventory.filter(i => i.status === 'Low Stock').length} 
          icon={Box} 
          color="bg-orange-500"
        />
        <StatCard 
          title="Out of Stock" 
          value={inventory.filter(i => i.status === 'Out of Stock').length} 
          icon={XCircle} 
          color="bg-red-500"
        />
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <select 
            value={statusFilter}
            onChange={(e) => setStatusFilter(e.target.value)}
            className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="all">All Status</option>
            <option value="In Stock">In Stock</option>
            <option value="Low Stock">Low Stock</option>
            <option value="Out of Stock">Out of Stock</option>
          </select>
          {(isAdmin || isManager) && (
            <>
              <button 
                onClick={() => { setEditingItem(null); setShowItemModal(true); }}
                className="flex items-center justify-center gap-2 px-4 py-2 bg-slate-100 dark:bg-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-200 dark:hover:bg-slate-600 transition-colors"
              >
                <Plus className="w-4 h-4" />
                <span>Add Item</span>
              </button>
              <button className="flex items-center justify-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
                <Download className="w-4 h-4" />
                <span>Export</span>
              </button>
            </>
          )}
        </div>
      </div>

      <DataTable 
        columns={inventoryColumns} 
        data={filteredInventory} 
        keyField="id" 
        onView={(item) => alert(`View: ${item.name}`)}
        onEdit={(item) => { setEditingItem(item); setShowItemModal(true); }}
        onDelete={(item) => {
          if (confirm(`Delete "${item.name}"?`)) {
            setInventory(inventory.filter(i => i.id !== item.id));
          }
        }}
        actions={['view', 'edit', 'delete']}
      />

      <Modal 
        title={editingItem ? 'Edit Inventory Item' : 'Add Inventory Item'} 
        isOpen={showItemModal && !showPurchaseModal} 
        onClose={() => { setShowItemModal(false); setEditingItem(null); }}
      >
        <form 
          onSubmit={(e) => {
            e.preventDefault();
            const formData = new FormData(e.target);
            const newItem = {
              id: editingItem?.id || inventory.length + 1,
              name: formData.get('name'),
              category: formData.get('category'),
              unit: formData.get('unit'),
              currentStock: parseFloat(formData.get('currentStock')) || 0,
              minStockLevel: parseFloat(formData.get('minStockLevel')) || 0,
              costPerUnit: parseFloat(formData.get('costPerUnit')) || 0,
              supplier: formData.get('supplier'),
              supplierPhone: formData.get('supplierPhone'),
              status: parseFloat(formData.get('currentStock')) >= parseFloat(formData.get('minStockLevel')) ? 'In Stock' : 
                     parseFloat(formData.get('currentStock')) > 0 ? 'Low Stock' : 'Out of Stock',
              lastUpdated: 'Just now'
            };
            
            if (editingItem) {
              setInventory(inventory.map(item => item.id === newItem.id ? newItem : item));
            } else {
              setInventory([...inventory, newItem]);
            }
            
            setShowItemModal(false);
            setEditingItem(null);
          }}
          className="space-y-4"
        >
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <FormInput 
              label="Item Name" 
              type="text" 
              value={editingItem?.name || ''} 
              onChange={(e) => setEditingItem({...editingItem, name: e.target.value})}
              placeholder="e.g., Paneer"
              required
            />
            <FormInput 
              label="Category" 
              type="text" 
              value={editingItem?.category || ''} 
              onChange={(e) => setEditingItem({...editingItem, category: e.target.value})}
              placeholder="e.g., Dairy"
              required
            />
          </div>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
            <FormInput 
              label="Unit" 
              type="text" 
              value={editingItem?.unit || ''} 
              onChange={(e) => setEditingItem({...editingItem, unit: e.target.value})}
              placeholder="e.g., kg"
              required
            />
            <FormInput 
              label="Current Stock" 
              type="number" 
              value={editingItem?.currentStock || ''} 
              onChange={(e) => setEditingItem({...editingItem, currentStock: parseFloat(e.target.value) || 0})}
              placeholder="5.2"
              required
            />
            <FormInput 
              label="Min Stock Level" 
              type="number" 
              value={editingItem?.minStockLevel || ''} 
              onChange={(e) => setEditingItem({...editingItem, minStockLevel: parseFloat(e.target.value) || 0})}
              placeholder="2"
              required
            />
          </div>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <FormInput 
              label="Cost per Unit (₹)" 
              type="number" 
              value={editingItem?.costPerUnit || ''} 
              onChange={(e) => setEditingItem({...editingItem, costPerUnit: parseFloat(e.target.value) || 0})}
              placeholder="450"
              required
            />
            <FormInput 
              label="Supplier" 
              type="text" 
              value={editingItem?.supplier || ''} 
              onChange={(e) => setEditingItem({...editingItem, supplier: e.target.value})}
              placeholder="ABC Dairy"
              required
            />
          </div>
          <FormInput 
            label="Supplier Phone" 
            type="text" 
            value={editingItem?.supplierPhone || ''} 
            onChange={(e) => setEditingItem({...editingItem, supplierPhone: e.target.value})}
            placeholder="+91 9876543210"
          />
          <div className="flex items-center justify-end gap-2">
            <button 
              type="button"
              onClick={() => { setShowItemModal(false); setEditingItem(null); }}
              className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
            >
              {editingItem ? 'Update Item' : 'Add Item'}
            </button>
          </div>
        </form>
      </Modal>

      <Modal 
        title={`Purchase ${editingItem?.name || ''}`} 
        isOpen={showPurchaseModal} 
        onClose={() => { setShowPurchaseModal(false); setEditingItem(null); }}
      >
        <form onSubmit={handlePurchaseSubmit} className="space-y-4">
          <div className="bg-slate-50 dark:bg-slate-700 rounded-lg p-4 mb-4">
            <div className="grid grid-cols-2 gap-4 text-sm">
              <div>
                <p className="text-slate-500 dark:text-slate-400">Current Stock</p>
                <p className="font-medium text-slate-800 dark:text-slate-100">{editingItem?.currentStock} {editingItem?.unit}</p>
              </div>
              <div>
                <p className="text-slate-500 dark:text-slate-400">Min Stock Level</p>
                <p className="font-medium text-slate-800 dark:text-slate-100">{editingItem?.minStockLevel} {editingItem?.unit}</p>
              </div>
              <div>
                <p className="text-slate-500 dark:text-slate-400">Cost per Unit</p>
                <p className="font-medium text-slate-800 dark:text-slate-100">₹{editingItem?.costPerUnit}</p>
              </div>
              <div>
                <p className="text-slate-500 dark:text-slate-400">Supplier</p>
                <p className="font-medium text-slate-800 dark:text-slate-100">{editingItem?.supplier}</p>
              </div>
            </div>
          </div>
          
          <FormInput 
            label="Quantity to Purchase" 
            type="number" 
            placeholder={`Enter quantity in ${editingItem?.unit}`}
            required
          />
          <div className="bg-slate-50 dark:bg-slate-700 rounded-lg p-3 text-sm">
            <p className="text-slate-500 dark:text-slate-400">
              Estimated Cost: ₹{(editingItem?.costPerUnit || 0)}
            </p>
          </div>
          <div className="flex items-center justify-end gap-2">
            <button 
              type="button"
              onClick={() => { setShowPurchaseModal(false); setEditingItem(null); }}
              className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              className="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors"
            >
              Record Purchase
            </button>
          </div>
        </form>
      </Modal>
    </div>
  );
}

// ============================================
// BILLING PAGE
// ============================================
function BillingPage() {
  const [invoices, setInvoices] = useState([
    { 
      id: 1, invoiceNumber: 'INV-20260726-001', orderNumber: 'ORD-20260726-001',
      customer: 'John Doe', customerEmail: 'john@example.com',
      amount: 2450, tax: 245, discount: 0, totalAmount: 2695,
      paymentStatus: 'Paid', paymentMethod: 'Card',
      date: '2026-07-26', dueDate: '2026-07-26', paidDate: '2026-07-26',
      notes: 'Paid via credit card'
    },
    { 
      id: 2, invoiceNumber: 'INV-20260726-002', orderNumber: 'ORD-20260726-002',
      customer: 'Priya Sharma', customerEmail: 'priya@example.com',
      amount: 1800, tax: 180, discount: 100, totalAmount: 1880,
      paymentStatus: 'Pending', paymentMethod: 'Cash',
      date: '2026-07-26', dueDate: '2026-07-27', paidDate: null,
      notes: 'Customer requested to pay later'
    },
  ]);
  const [paymentStatusFilter, setPaymentStatusFilter] = useState('all');
  const [showInvoiceModal, setShowInvoiceModal] = useState(false);
  const [showPaymentModal, setShowPaymentModal] = useState(false);
  const [selectedInvoice, setSelectedInvoice] = useState(null);

  const paymentStatusOptions = ['All', 'Paid', 'Partial', 'Unpaid', 'Refunded'];

  const filteredInvoices = invoices.filter(invoice => {
    const matchesStatus = paymentStatusFilter === 'all' || invoice.paymentStatus.toLowerCase() === paymentStatusFilter.toLowerCase();
    return matchesStatus;
  });

  const invoiceColumns = [
    { key: 'invoiceNumber', label: 'Invoice #' },
    { key: 'orderNumber', label: 'Order #' },
    { key: 'customer', label: 'Customer' },
    { 
      key: 'totalAmount', 
      label: 'Total Amount',
      render: (value) => `₹${value.toLocaleString()}`
    },
    { 
      key: 'paymentStatus', 
      label: 'Status',
      render: (value) => (
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
          value === 'Paid' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
          value === 'Partial' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
          value === 'Unpaid' ? 'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300' :
          'bg-slate-100 text-slate-700 dark:bg-slate-900 dark:text-slate-300'
        }`}>
          {value}
        </span>
      )
    },
    { key: 'paymentMethod', label: 'Method' },
    { key: 'date', label: 'Date' },
    { key: 'dueDate', label: 'Due Date' }
  ];

  const handleViewInvoice = (invoice) => {
    setSelectedInvoice(invoice);
    setShowInvoiceModal(true);
  };

  const handleRecordPayment = (invoice) => {
    setSelectedInvoice(invoice);
    setShowPaymentModal(true);
  };

  const handlePaymentSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const amount = parseFloat(formData.get('amount')) || 0;
    const paymentMethod = formData.get('paymentMethod');
    const notes = formData.get('notes');
    
    setInvoices(invoices.map(invoice => {
      if (invoice.id === selectedInvoice?.id) {
        return {
          ...invoice,
          paymentStatus: 'Paid',
          paidDate: new Date().toISOString().split('T')[0],
          notes: notes || invoice.notes
        };
      }
      return invoice;
    }));
    
    setShowPaymentModal(false);
    setSelectedInvoice(null);
    alert(`Payment of ₹${amount} recorded successfully!`);
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Billing</h1>
          <p className="text-slate-500 dark:text-slate-400 mt-1">Manage invoices and payments</p>
        </div>
        <button className="flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
          <Plus className="w-4 h-4" />
          <span>New Invoice</span>
        </button>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <StatCard 
          title="Total Invoices" 
          value={invoices.length} 
          icon={FileText} 
          color="bg-blue-500"
        />
        <StatCard 
          title="Paid Amount" 
          value={`₹${invoices.filter(i => i.paymentStatus === 'Paid').reduce((sum, i) => sum + i.totalAmount, 0).toLocaleString()}`} 
          icon={CheckCircle} 
          color="bg-green-500"
        />
        <StatCard 
          title="Pending Amount" 
          value={`₹${invoices.filter(i => i.paymentStatus !== 'Paid').reduce((sum, i) => sum + i.totalAmount, 0).toLocaleString()}`} 
          icon={Clock} 
          color="bg-orange-500"
        />
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
          <select 
            value={paymentStatusFilter}
            onChange={(e) => setPaymentStatusFilter(e.target.value)}
            className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            {paymentStatusOptions.map(option => (
              <option key={option.toLowerCase()} value={option.toLowerCase()}>
                {option}
              </option>
            ))}
          </select>
          <select className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500">
            <option>This Week</option>
            <option>This Month</option>
            <option>This Year</option>
          </select>
          <button className="flex items-center justify-center gap-2 px-4 py-2 bg-slate-100 dark:bg-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-200 dark:hover:bg-slate-600 transition-colors">
            <Download className="w-4 h-4" />
            <span>Export</span>
          </button>
        </div>
      </div>

      <DataTable 
        columns={invoiceColumns} 
        data={filteredInvoices} 
        keyField="id" 
        onView={handleViewInvoice}
        onEdit={(invoice) => alert(`Edit invoice: ${invoice.invoiceNumber}`)}
        actions={['view', 'edit']}
      />

      <Modal 
        title={`Invoice #${selectedInvoice?.invoiceNumber || ''}`} 
        isOpen={showInvoiceModal} 
        onClose={() => setShowInvoiceModal(false)}
        size="lg"
      >
        {selectedInvoice && (
          <div className="space-y-4">
            <div className="bg-slate-50 dark:bg-slate-700 rounded-lg p-4">
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                  <h3 className="font-semibold text-slate-800 dark:text-slate-100">From:</h3>
                  <p className="text-sm text-slate-600 dark:text-slate-300 mt-1">My Restaurant</p>
                  <p className="text-sm text-slate-500 dark:text-slate-400">123 Main Street, Mumbai</p>
                </div>
                <div>
                  <h3 className="font-semibold text-slate-800 dark:text-slate-100">To:</h3>
                  <p className="text-sm text-slate-600 dark:text-slate-300 mt-1">{selectedInvoice.customer}</p>
                  <p className="text-sm text-slate-500 dark:text-slate-400">{selectedInvoice.customerEmail}</p>
                </div>
                <div className="text-right">
                  <h3 className="font-semibold text-slate-800 dark:text-slate-100">Invoice Details</h3>
                  <p className="text-sm text-slate-600 dark:text-slate-300 mt-1">#{selectedInvoice.invoiceNumber}</p>
                  <p className="text-sm text-slate-500 dark:text-slate-400">Order: {selectedInvoice.orderNumber}</p>
                  <p className="text-sm text-slate-500 dark:text-slate-400">Date: {selectedInvoice.date}</p>
                </div>
              </div>
            </div>

            <div>
              <h3 className="font-semibold text-slate-800 dark:text-slate-100 mb-3">Items</h3>
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead>
                    <tr className="border-b border-slate-200 dark:border-slate-700">
                      <th className="text-left py-2 px-3 text-sm font-medium text-slate-500 dark:text-slate-400">#</th>
                      <th className="text-left py-2 px-3 text-sm font-medium text-slate-500 dark:text-slate-400">Item</th>
                      <th className="text-left py-2 px-3 text-sm font-medium text-slate-500 dark:text-slate-400">Qty</th>
                      <th className="text-right py-2 px-3 text-sm font-medium text-slate-500 dark:text-slate-400">Price</th>
                      <th className="text-right py-2 px-3 text-sm font-medium text-slate-500 dark:text-slate-400">Amount</th>
                    </tr>
                  </thead>
                  <tbody>
                    {[1, 2, 3].map((item, index) => (
                      <tr key={index} className="border-b border-slate-100 dark:border-slate-700">
                        <td className="py-2 px-3 text-sm text-slate-800 dark:text-slate-100">{index + 1}</td>
                        <td className="py-2 px-3 text-sm text-slate-600 dark:text-slate-300">
                          {index === 0 ? 'Paneer Tikka Masala' : index === 1 ? 'Jeera Rice' : 'Raita'}
                        </td>
                        <td className="py-2 px-3 text-sm text-slate-500 dark:text-slate-400">
                          {index === 0 ? '2' : '1'}
                        </td>
                        <td className="py-2 px-3 text-sm text-right text-slate-800 dark:text-slate-100">
                          ₹{index === 0 ? '450' : index === 1 ? '200' : '80'}
                        </td>
                        <td className="py-2 px-3 text-sm text-right text-green-600">
                          ₹{index === 0 ? '900' : index === 1 ? '200' : '80'}
                        </td>
                      </tr>
                    ))}
                  </tbody>
                  <tfoot>
                    <tr className="border-t border-slate-200 dark:border-slate-700">
                      <td colSpan={4} className="py-2 px-3 text-right text-sm font-medium text-slate-800 dark:text-slate-100">
                        Subtotal:
                      </td>
                      <td className="py-2 px-3 text-sm text-right text-slate-800 dark:text-slate-100">
                        ₹{selectedInvoice.amount.toLocaleString()}
                      </td>
                    </tr>
                    <tr>
                      <td colSpan={4} className="py-2 px-3 text-right text-sm font-medium text-slate-800 dark:text-slate-100">
                        Discount:
                      </td>
                      <td className="py-2 px-3 text-sm text-right text-green-600">
                        -₹{selectedInvoice.discount.toLocaleString()}
                      </td>
                    </tr>
                    <tr>
                      <td colSpan={4} className="py-2 px-3 text-right text-sm font-medium text-slate-800 dark:text-slate-100">
                        Tax:
                      </td>
                      <td className="py-2 px-3 text-sm text-right text-slate-800 dark:text-slate-100">
                        ₹{selectedInvoice.tax.toLocaleString()}
                      </td>
                    </tr>
                    <tr className="bg-slate-50 dark:bg-slate-700">
                      <td colSpan={4} className="py-2 px-3 text-right text-sm font-bold text-slate-800 dark:text-slate-100">
                        Total:
                      </td>
                      <td className="py-2 px-3 text-sm text-right font-bold text-blue-600">
                        ₹{selectedInvoice.totalAmount.toLocaleString()}
                      </td>
                    </tr>
                  </tfoot>
                </table>
              </div>
            </div>

            <div className="bg-slate-50 dark:bg-slate-700 rounded-lg p-4">
              <h3 className="font-semibold text-slate-800 dark:text-slate-100 mb-3">Payment Information</h3>
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <p className="text-sm text-slate-500 dark:text-slate-400">Payment Status</p>
                  <span className={`inline-block px-3 py-1 rounded-full text-sm font-medium mt-1 ${
                    selectedInvoice.paymentStatus === 'Paid' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
                    selectedInvoice.paymentStatus === 'Partial' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
                    'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300'
                  }`}>
                    {selectedInvoice.paymentStatus}
                  </span>
                </div>
                <div>
                  <p className="text-sm text-slate-500 dark:text-slate-400">Payment Method</p>
                  <p className="text-sm font-medium text-slate-800 dark:text-slate-100 mt-1">{selectedInvoice.paymentMethod}</p>
                </div>
                <div>
                  <p className="text-sm text-slate-500 dark:text-slate-400">Due Date</p>
                  <p className="text-sm font-medium text-slate-800 dark:text-slate-100 mt-1">{selectedInvoice.dueDate}</p>
                </div>
                <div>
                  <p className="text-sm text-slate-500 dark:text-slate-400">Paid Date</p>
                  <p className="text-sm font-medium text-slate-800 dark:text-slate-100 mt-1">
                    {selectedInvoice.paidDate || 'Not paid yet'}
                  </p>
                </div>
              </div>
            </div>

            <div className="flex items-center justify-end gap-2 pt-4 border-t border-slate-200 dark:border-slate-700">
              <button className="flex items-center gap-2 px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors">
                <Printer className="w-4 h-4" />
                <span>Print</span>
              </button>
              <button className="flex items-center gap-2 px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors">
                <FileText className="w-4 h-4" />
                <span>Copy Invoice #</span>
              </button>
              {selectedInvoice.paymentStatus !== 'Paid' && (
                <button 
                  onClick={() => { setShowInvoiceModal(false); handleRecordPayment(selectedInvoice); }}
                  className="flex items-center gap-2 px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors"
                >
                  <CreditCard className="w-4 h-4" />
                  <span>Record Payment</span>
                </button>
              )}
            </div>
          </div>
        )}
      </Modal>

      <Modal 
        title={`Record Payment for Invoice #${selectedInvoice?.invoiceNumber || ''}`} 
        isOpen={showPaymentModal} 
        onClose={() => { setShowPaymentModal(false); setSelectedInvoice(null); }}
      >
        {selectedInvoice && (
          <form onSubmit={handlePaymentSubmit} className="space-y-4">
            <div className="bg-slate-50 dark:bg-slate-700 rounded-lg p-4 mb-4">
              <div className="grid grid-cols-2 gap-4 text-sm">
                <div>
                  <p className="text-slate-500 dark:text-slate-400">Invoice Total</p>
                  <p className="font-medium text-slate-800 dark:text-slate-100">₹{selectedInvoice.totalAmount.toLocaleString()}</p>
                </div>
                <div>
                  <p className="text-slate-500 dark:text-slate-400">Amount Paid</p>
                  <p className="font-medium text-slate-800 dark:text-slate-100">₹0</p>
                </div>
              </div>
            </div>

            <FormInput 
              label="Payment Amount (₹)" 
              type="number" 
              placeholder={`Max: ₹${selectedInvoice.totalAmount.toLocaleString()}`}
              required
            />
            <FormInput 
              label="Payment Method" 
              type="select" 
              required
              options={[
                { value: 'Cash', label: 'Cash' },
                { value: 'Card', label: 'Credit/Debit Card' },
                { value: 'Online', label: 'Online Transfer' },
                { value: 'UPI', label: 'UPI Payment' }
              ]}
            />
            <FormInput 
              label="Notes" 
              type="textarea" 
              placeholder="Additional payment notes..."
            />
            <div className="flex items-center justify-end gap-2">
              <button 
                type="button"
                onClick={() => { setShowPaymentModal(false); setSelectedInvoice(null); }}
                className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors"
              >
                Cancel
              </button>
              <button 
                type="submit"
                className="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors"
              >
                Record Payment
              </button>
            </div>
          </form>
        )}
      </Modal>
    </div>
  );
}

// ============================================
// STAFF PAGE
// ============================================
function StaffPage() {
  const { isAdmin, isManager } = useContext(AuthContext);
  const [staff, setStaff] = useState([
    { 
      id: 1, userId: 1, firstName: 'Admin', lastName: 'User', email: 'admin@example.com',
      phone: '+91 9876543210', role: 'Admin', isActive: true, 
      joinDate: '2024-01-01', lastLogin: '2026-07-26 10:00 AM'
    },
    { 
      id: 2, userId: 2, firstName: 'John', lastName: 'Manager', email: 'manager@example.com',
      phone: '+91 9876543211', role: 'Manager', isActive: true,
      joinDate: '2024-02-15', lastLogin: '2026-07-26 09:30 AM'
    },
    { 
      id: 3, userId: 3, firstName: 'Priya', lastName: 'Sharma', email: 'priya@example.com',
      phone: '+91 9876543212', role: 'Staff', isActive: true,
      joinDate: '2024-03-01', lastLogin: '2026-07-26 09:00 AM'
    },
  ]);
  const [roles] = useState([
    { id: 1, name: 'Admin', description: 'Full access to all features' },
    { id: 2, name: 'Manager', description: 'Manage staff, inventory, and reports' },
    { id: 3, name: 'Staff', description: 'Take orders, manage tables' }
  ]);
  const [showStaffModal, setShowStaffModal] = useState(false);
  const [editingStaff, setEditingStaff] = useState(null);

  const staffColumns = [
    { 
      key: 'fullName', 
      label: 'Name',
      render: (value, row) => `${row.firstName} ${row.lastName}`
    },
    { key: 'email', label: 'Email' },
    { key: 'phone', label: 'Phone' },
    { 
      key: 'role', 
      label: 'Role',
      render: (value) => (
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
          value === 'Admin' ? 'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300' :
          value === 'Manager' ? 'bg-blue-100 text-blue-700 dark:bg-blue-900 dark:text-blue-300' :
          'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300'
        }`}>
          {value}
        </span>
      )
    },
    { 
      key: 'isActive', 
      label: 'Status',
      render: (value) => (
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
          value ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
          'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300'
        }`}>
          {value ? 'Active' : 'Inactive'}
        </span>
      )
    },
    { key: 'joinDate', label: 'Join Date' },
    { key: 'lastLogin', label: 'Last Login' }
  ];

  const handleStaffSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const newStaff = {
      id: editingStaff?.id || staff.length + 1,
      userId: editingStaff?.userId || staff.length + 1,
      firstName: formData.get('firstName'),
      lastName: formData.get('lastName'),
      email: formData.get('email'),
      phone: formData.get('phone'),
      role: formData.get('role'),
      isActive: formData.get('isActive') === 'on',
      joinDate: editingStaff?.joinDate || new Date().toISOString().split('T')[0],
      lastLogin: editingStaff?.lastLogin || 'Never'
    };
    
    if (editingStaff) {
      setStaff(staff.map(s => s.id === newStaff.id ? newStaff : s));
    } else {
      setStaff([...staff, newStaff]);
    }
    
    setShowStaffModal(false);
    setEditingStaff(null);
    alert(`Staff member ${editingStaff ? 'updated' : 'added'} successfully!`);
  };

  const handleDeleteStaff = (member) => {
    if (confirm(`Delete staff member "${member.firstName} ${member.lastName}"?`)) {
      setStaff(staff.filter(s => s.id !== member.id));
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Staff Management</h1>
          <p className="text-slate-500 dark:text-slate-400 mt-1">Manage your team members</p>
        </div>
        {isAdmin && (
          <button 
            onClick={() => { setEditingStaff(null); setShowStaffModal(true); }}
            className="flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
          >
            <Plus className="w-4 h-4" />
            <span>Add Staff</span>
          </button>
        )}
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <StatCard 
          title="Total Staff" 
          value={staff.length} 
          icon={Users} 
          color="bg-blue-500"
        />
        <StatCard 
          title="Active Staff" 
          value={staff.filter(s => s.isActive).length} 
          icon={CheckCircle} 
          color="bg-green-500"
        />
        <StatCard 
          title="Inactive Staff" 
          value={staff.filter(s => !s.isActive).length} 
          icon={XCircle} 
          color="bg-red-500"
        />
      </div>

      <DataTable 
        columns={staffColumns} 
        data={staff} 
        keyField="id" 
        onView={(member) => alert(`View: ${member.firstName} ${member.lastName}`)}
        onEdit={(member) => { setEditingStaff(member); setShowStaffModal(true); }}
        onDelete={isAdmin ? handleDeleteStaff : null}
        actions={isAdmin ? ['view', 'edit', 'delete'] : ['view']}
      />

      <Modal 
        title={editingStaff ? 'Edit Staff Member' : 'Add Staff Member'} 
        isOpen={showStaffModal} 
        onClose={() => { setShowStaffModal(false); setEditingStaff(null); }}
      >
        <form onSubmit={handleStaffSubmit} className="space-y-4">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <FormInput 
              label="First Name" 
              type="text" 
              value={editingStaff?.firstName || ''} 
              onChange={(e) => setEditingStaff({...editingStaff, firstName: e.target.value})}
              placeholder="John"
              required
            />
            <FormInput 
              label="Last Name" 
              type="text" 
              value={editingStaff?.lastName || ''} 
              onChange={(e) => setEditingStaff({...editingStaff, lastName: e.target.value})}
              placeholder="Doe"
              required
            />
          </div>
          <FormInput 
            label="Email Address" 
            type="email" 
            value={editingStaff?.email || ''} 
            onChange={(e) => setEditingStaff({...editingStaff, email: e.target.value})}
            placeholder="john@example.com"
            required
          />
          <FormInput 
            label="Phone Number" 
            type="tel" 
            value={editingStaff?.phone || ''} 
            onChange={(e) => setEditingStaff({...editingStaff, phone: e.target.value})}
            placeholder="+91 9876543210"
          />
          <FormInput 
            label="Role" 
            type="select" 
            value={editingStaff?.role || ''} 
            onChange={(e) => setEditingStaff({...editingStaff, role: e.target.value})}
            placeholder="Select role"
            required
            options={roles.map(r => ({ value: r.name, label: r.name }))}
          />
          <label className="flex items-center gap-2 cursor-pointer mb-4">
            <input 
              type="checkbox" 
              checked={editingStaff?.isActive || true}
              onChange={(e) => setEditingStaff({...editingStaff, isActive: e.target.checked})}
              className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
            />
            <span className="text-sm text-slate-700 dark:text-slate-300">Active</span>
          </label>
          <div className="flex items-center justify-end gap-2">
            <button 
              type="button"
              onClick={() => { setShowStaffModal(false); setEditingStaff(null); }}
              className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
            >
              {editingStaff ? 'Update Staff' : 'Add Staff'}
            </button>
          </div>
        </form>
      </Modal>
    </div>
  );
}

// ============================================
// CUSTOMERS PAGE
// ============================================
function CustomersPage() {
  const { isAdmin, isManager } = useContext(AuthContext);
  const [customers, setCustomers] = useState([
    { 
      id: 1, userId: 101, firstName: 'John', lastName: 'Doe', email: 'john@example.com',
      phone: '+91 9876543210', address: '123 Main Street, Mumbai',
      loyaltyPoints: 1250, totalSpent: 15000, totalOrders: 12,
      preferredCuisine: ['North Indian', 'Chinese'],
      allergies: ['Peanuts'],
      joinDate: '2024-01-01', lastOrderDate: '2026-07-25'
    },
    { 
      id: 2, userId: 102, firstName: 'Priya', lastName: 'Sharma', email: 'priya@example.com',
      phone: '+91 9876543211', address: '456 Park Avenue, Delhi',
      loyaltyPoints: 850, totalSpent: 8500, totalOrders: 8,
      preferredCuisine: ['South Indian', 'Italian'],
      allergies: [],
      joinDate: '2024-02-15', lastOrderDate: '2026-07-26'
    },
  ]);
  const [loyaltyTiers] = useState([
    { name: 'Bronze', minPoints: 0, maxPoints: 999, discount: '5%' },
    { name: 'Silver', minPoints: 1000, maxPoints: 2999, discount: '10%' },
    { name: 'Gold', minPoints: 3000, maxPoints: 4999, discount: '15%' },
    { name: 'Platinum', minPoints: 5000, maxPoints: Infinity, discount: '20%' }
  ]);
  const [showCustomerModal, setShowCustomerModal] = useState(false);
  const [editingCustomer, setEditingCustomer] = useState(null);

  const customerColumns = [
    { 
      key: 'fullName', 
      label: 'Name',
      render: (value, row) => `${row.firstName} ${row.lastName}`
    },
    { key: 'email', label: 'Email' },
    { key: 'phone', label: 'Phone' },
    { 
      key: 'totalOrders', 
      label: 'Orders',
      render: (value) => value
    },
    { 
      key: 'totalSpent', 
      label: 'Total Spent',
      render: (value) => `₹${value.toLocaleString()}`
    },
    { 
      key: 'loyaltyPoints', 
      label: 'Loyalty Points',
      render: (value) => value
    },
    { 
      key: 'tier', 
      label: 'Tier',
      render: (value, row) => {
        const tier = loyaltyTiers.find(t => row.loyaltyPoints >= t.minPoints && row.loyaltyPoints <= t.maxPoints);
        return (
          <span className={`px-2 py-1 rounded-full text-xs font-medium ${
            tier?.name === 'Bronze' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
            tier?.name === 'Silver' ? 'bg-slate-100 text-slate-700 dark:bg-slate-900 dark:text-slate-300' :
            tier?.name === 'Gold' ? 'bg-yellow-100 text-yellow-700 dark:bg-yellow-900 dark:text-yellow-300' :
            'bg-purple-100 text-purple-700 dark:bg-purple-900 dark:text-purple-300'
          }`}>
            {tier?.name || 'Bronze'}
          </span>
        );
      }
    },
    { key: 'lastOrderDate', label: 'Last Order' }
  ];

  const handleCustomerSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const newCustomer = {
      id: editingCustomer?.id || customers.length + 1,
      userId: editingCustomer?.userId || customers.length + 101,
      firstName: formData.get('firstName'),
      lastName: formData.get('lastName'),
      email: formData.get('email'),
      phone: formData.get('phone'),
      address: formData.get('address'),
      loyaltyPoints: parseInt(formData.get('loyaltyPoints')) || 0,
      totalSpent: parseFloat(formData.get('totalSpent')) || 0,
      totalOrders: parseInt(formData.get('totalOrders')) || 0,
      preferredCuisine: formData.getAll('preferredCuisine').filter(x => x),
      allergies: formData.getAll('allergies').filter(x => x),
      joinDate: editingCustomer?.joinDate || new Date().toISOString().split('T')[0],
      lastOrderDate: editingCustomer?.lastOrderDate || 'Never'
    };
    
    if (editingCustomer) {
      setCustomers(customers.map(c => c.id === newCustomer.id ? newCustomer : c));
    } else {
      setCustomers([...customers, newCustomer]);
    }
    
    setShowCustomerModal(false);
    setEditingCustomer(null);
    alert(`Customer ${editingCustomer ? 'updated' : 'added'} successfully!`);
  };

  const handleDeleteCustomer = (customer) => {
    if (confirm(`Delete customer "${customer.firstName} ${customer.lastName}"?`)) {
      setCustomers(customers.filter(c => c.id !== customer.id));
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Customers</h1>
          <p className="text-slate-500 dark:text-slate-400 mt-1">Manage customer information and loyalty</p>
        </div>
        {(isAdmin || isManager) && (
          <button 
            onClick={() => { setEditingCustomer(null); setShowCustomerModal(true); }}
            className="flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
          >
            <Plus className="w-4 h-4" />
            <span>Add Customer</span>
          </button>
        )}
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <StatCard 
          title="Total Customers" 
          value={customers.length} 
          icon={Users} 
          color="bg-blue-500"
        />
        <StatCard 
          title="Total Orders" 
          value={customers.reduce((sum, c) => sum + c.totalOrders, 0)} 
          icon={ShoppingCart} 
          color="bg-green-500"
        />
        <StatCard 
          title="Total Revenue" 
          value={`₹${customers.reduce((sum, c) => sum + c.totalSpent, 0).toLocaleString()}`} 
          icon={DollarSign} 
          color="bg-purple-500"
        />
      </div>

      <DataTable 
        columns={customerColumns} 
        data={customers} 
        keyField="id" 
        onView={(customer) => alert(`View: ${customer.firstName} ${customer.lastName}`)}
        onEdit={(customer) => { setEditingCustomer(customer); setShowCustomerModal(true); }}
        onDelete={(isAdmin || isManager) ? handleDeleteCustomer : null}
        actions={(isAdmin || isManager) ? ['view', 'edit', 'delete'] : ['view']}
      />

      <Modal 
        title={editingCustomer ? 'Edit Customer' : 'Add Customer'} 
        isOpen={showCustomerModal} 
        onClose={() => { setShowCustomerModal(false); setEditingCustomer(null); }}
        size="lg"
      >
        <form onSubmit={handleCustomerSubmit} className="space-y-4">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <FormInput 
              label="First Name" 
              type="text" 
              value={editingCustomer?.firstName || ''} 
              onChange={(e) => setEditingCustomer({...editingCustomer, firstName: e.target.value})}
              placeholder="John"
              required
            />
            <FormInput 
              label="Last Name" 
              type="text" 
              value={editingCustomer?.lastName || ''} 
              onChange={(e) => setEditingCustomer({...editingCustomer, lastName: e.target.value})}
              placeholder="Doe"
              required
            />
          </div>
          <FormInput 
            label="Email Address" 
            type="email" 
            value={editingCustomer?.email || ''} 
            onChange={(e) => setEditingCustomer({...editingCustomer, email: e.target.value})}
            placeholder="john@example.com"
          />
          <FormInput 
            label="Phone Number" 
            type="tel" 
            value={editingCustomer?.phone || ''} 
            onChange={(e) => setEditingCustomer({...editingCustomer, phone: e.target.value})}
            placeholder="+91 9876543210"
          />
          <FormInput 
            label="Address" 
            type="textarea" 
            value={editingCustomer?.address || ''} 
            onChange={(e) => setEditingCustomer({...editingCustomer, address: e.target.value})}
            placeholder="123 Main Street, City"
          />
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <FormInput 
              label="Loyalty Points" 
              type="number" 
              value={editingCustomer?.loyaltyPoints || ''} 
              onChange={(e) => setEditingCustomer({...editingCustomer, loyaltyPoints: parseInt(e.target.value) || 0})}
              placeholder="1250"
            />
            <FormInput 
              label="Total Spent (₹)" 
              type="number" 
              value={editingCustomer?.totalSpent || ''} 
              onChange={(e) => setEditingCustomer({...editingCustomer, totalSpent: parseFloat(e.target.value) || 0})}
              placeholder="15000"
            />
          </div>
          <FormInput 
            label="Total Orders" 
            type="number" 
            value={editingCustomer?.totalOrders || ''} 
            onChange={(e) => setEditingCustomer({...editingCustomer, totalOrders: parseInt(e.target.value) || 0})}
            placeholder="12"
          />
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <div>
              <label className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">
                Preferred Cuisine
              </label>
              <div className="space-y-2">
                {['North Indian', 'South Indian', 'Chinese', 'Italian'].map(cuisine => (
                  <label key={cuisine} className="flex items-center gap-2 cursor-pointer">
                    <input 
                      type="checkbox" 
                      checked={(editingCustomer?.preferredCuisine || []).includes(cuisine)}
                      onChange={(e) => {
                        const currentCuisines = editingCustomer?.preferredCuisine || [];
                        setEditingCustomer({
                          ...editingCustomer,
                          preferredCuisine: e.target.checked 
                            ? [...currentCuisines, cuisine] 
                            : currentCuisines.filter(c => c !== cuisine)
                        });
                      }}
                      className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
                    />
                    <span className="text-sm text-slate-700 dark:text-slate-300">{cuisine}</span>
                  </label>
                ))}
              </div>
            </div>
            <div>
              <label className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">
                Allergies
              </label>
              <div className="space-y-2">
                {['Peanuts', 'Shellfish', 'Gluten', 'Dairy'].map(allergy => (
                  <label key={allergy} className="flex items-center gap-2 cursor-pointer">
                    <input 
                      type="checkbox" 
                      checked={(editingCustomer?.allergies || []).includes(allergy)}
                      onChange={(e) => {
                        const currentAllergies = editingCustomer?.allergies || [];
                        setEditingCustomer({
                          ...editingCustomer,
                          allergies: e.target.checked 
                            ? [...currentAllergies, allergy] 
                            : currentAllergies.filter(a => a !== allergy)
                        });
                      }}
                      className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
                    />
                    <span className="text-sm text-slate-700 dark:text-slate-300">{allergy}</span>
                  </label>
                ))}
              </div>
            </div>
          </div>
          <div className="flex items-center justify-end gap-2">
            <button 
              type="button"
              onClick={() => { setShowCustomerModal(false); setEditingCustomer(null); }}
              className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
            >
              {editingCustomer ? 'Update Customer' : 'Add Customer'}
            </button>
          </div>
        </form>
      </Modal>
    </div>
  );
}

// ============================================
// REPORTS PAGE
// ============================================
function ReportsPage() {
  const [reportType, setReportType] = useState('sales');
  const [dateRange, setDateRange] = useState('today');

  const salesData = {
    daily: [
      { date: '2026-07-26', orders: 128, revenue: 128000, discount: 5000, tax: 12800, net: 135800 },
      { date: '2026-07-25', orders: 115, revenue: 115000, discount: 4000, tax: 11500, net: 122500 },
      { date: '2026-07-24', orders: 132, revenue: 132000, discount: 6000, tax: 13200, net: 139200 },
    ],
    weekly: [
      { week: 'Week 1', orders: 620, revenue: 620000, discount: 25000, tax: 62000, net: 657000 },
      { week: 'Week 2', orders: 710, revenue: 710000, discount: 30000, tax: 71000, net: 751000 },
    ],
    monthly: [
      { month: 'Jan', orders: 2800, revenue: 2800000, discount: 120000, tax: 280000, net: 2960000 },
      { month: 'Feb', orders: 2600, revenue: 2600000, discount: 110000, tax: 260000, net: 2750000 },
    ]
  };

  const expenseData = {
    categories: [
      { category: 'Rent', amount: 150000, percentage: 25 },
      { category: 'Salaries', amount: 250000, percentage: 42 },
      { category: 'Supplies', amount: 100000, percentage: 17 },
      { category: 'Utilities', amount: 40000, percentage: 7 },
      { category: 'Other', amount: 50000, percentage: 8 },
    ],
    monthly: [
      { month: 'Jan', amount: 500000, count: 45 },
      { month: 'Feb', amount: 480000, count: 42 },
    ]
  };

  const getReportData = () => {
    switch (reportType) {
      case 'sales':
        return dateRange === 'week' ? salesData.weekly : salesData.monthly;
      case 'expenses':
        return expenseData.monthly;
      default:
        return [];
    }
  };

  const reportColumns = {
    sales: [
      { key: 'date', label: reportType === 'sales' && dateRange === 'month' ? 'Month' : 'Date' },
      { key: 'orders', label: 'Orders' },
      { key: 'revenue', label: 'Revenue', render: (value) => `₹${value.toLocaleString()}` },
      { key: 'discount', label: 'Discount', render: (value) => `-₹${value.toLocaleString()}` },
      { key: 'tax', label: 'Tax', render: (value) => `₹${value.toLocaleString()}` },
      { key: 'net', label: 'Net Revenue', render: (value) => `₹${value.toLocaleString()}` }
    ],
    expenses: [
      { key: 'month', label: 'Month' },
      { key: 'amount', label: 'Amount', render: (value) => `₹${value.toLocaleString()}` },
      { key: 'count', label: 'Transactions' }
    ]
  };

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Reports</h1>
        <p className="text-slate-500 dark:text-slate-400 mt-1">Analyze your restaurant performance</p>
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
          <select 
            value={reportType}
            onChange={(e) => setReportType(e.target.value)}
            className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="sales">Sales Report</option>
            <option value="expenses">Expense Report</option>
          </select>
          <select 
            value={dateRange}
            onChange={(e) => setDateRange(e.target.value)}
            className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="today">Today</option>
            <option value="week">This Week</option>
            <option value="month">This Month</option>
          </select>
          <button className="flex items-center justify-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors">
            <Filter className="w-4 h-4" />
            <span>Apply Filters</span>
          </button>
        </div>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        {reportType === 'sales' && (
          <>
            <StatCard 
              title="Total Orders" 
              value={getReportData().reduce((sum, d) => sum + d.orders, 0)} 
              icon={ShoppingCart} 
              color="bg-blue-500"
            />
            <StatCard 
              title="Total Revenue" 
              value={`₹${getReportData().reduce((sum, d) => sum + d.revenue, 0).toLocaleString()}`} 
              icon={DollarSign} 
              color="bg-green-500"
            />
            <StatCard 
              title="Total Discount" 
              value={`-₹${getReportData().reduce((sum, d) => sum + d.discount, 0).toLocaleString()}`} 
              icon={Percent} 
              color="bg-orange-500"
            />
            <StatCard 
              title="Net Revenue" 
              value={`₹${getReportData().reduce((sum, d) => sum + d.net, 0).toLocaleString()}`} 
              icon={TrendingUp} 
              color="bg-purple-500"
            />
          </>
        )}
        {reportType === 'expenses' && (
          <>
            <StatCard 
              title="Total Expenses" 
              value={`₹${expenseData.monthly.reduce((sum, d) => sum + d.amount, 0).toLocaleString()}`} 
              icon={DollarSign} 
              color="bg-red-500"
            />
            <StatCard 
              title="Average Expense" 
              value={`₹${(expenseData.monthly.reduce((sum, d) => sum + d.amount, 0) / expenseData.monthly.length).toLocaleString()}`} 
              icon={BarChart} 
              color="bg-orange-500"
            />
            <StatCard 
              title="Highest Category" 
              value={expenseData.categories.reduce((max, c) => c.amount > max.amount ? c : max).category} 
              icon={Tag} 
              color="bg-purple-500"
            />
            <StatCard 
              title="Total Transactions" 
              value={expenseData.monthly.reduce((sum, d) => sum + d.count, 0)} 
              icon={Receipt} 
              color="bg-blue-500"
            />
          </>
        )}
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">
            {reportType === 'sales' ? 'Revenue Trend' : 'Expense Distribution'}
          </h2>
          <div className="h-64">
            {reportType === 'sales' && (
              <div className="h-full flex items-end gap-2">
                {getReportData().map((data, index) => (
                  <div key={index} className="flex-1 flex flex-col items-center gap-2">
                    <div 
                      className="w-full bg-blue-500 rounded-t" 
                      style={{ height: `${data.net / 10000}%` }}
                      title={`₹${data.net.toLocaleString()}`}
                    ></div>
                    <span className="text-xs text-slate-500 dark:text-slate-400">{data.date || data.month || data.week}</span>
                  </div>
                ))}
              </div>
            )}
            {reportType === 'expenses' && (
              <div className="h-full flex items-center justify-center">
                <div className="relative w-48 h-48">
                  {expenseData.categories.map((category, index) => {
                    const startAngle = index * (360 / expenseData.categories.length);
                    const endAngle = startAngle + (360 * category.percentage / 100);
                    return (
                      <div 
                        key={category.category}
                        className="absolute top-0 left-0 w-full h-full"
                        style={{
                          clipPath: `polygon(50% 50%, 50% 0%, ${50 + 48 * Math.cos((startAngle - 90) * Math.PI / 180)}% ${50 + 48 * Math.sin((startAngle - 90) * Math.PI / 180)}%, ${50 + 48 * Math.cos((endAngle - 90) * Math.PI / 180)}% ${50 + 48 * Math.sin((endAngle - 90) * Math.PI / 180)}%)`,
                          backgroundColor: ['#ef4444', '#f97316', '#eab308', '#22c55e', '#3b82f6'][index]
                        }}
                        title={`${category.category}: ₹${category.amount.toLocaleString()} (${category.percentage}%)`}
                      ></div>
                    );
                  })}
                  <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-center">
                    <p className="text-lg font-bold text-slate-800 dark:text-slate-100">Total</p>
                    <p className="text-sm text-slate-500 dark:text-slate-400">₹{expenseData.categories.reduce((sum, c) => sum + c.amount, 0).toLocaleString()}</p>
                  </div>
                </div>
              </div>
            )}
          </div>
        </div>

        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">
            {reportType === 'sales' ? 'Daily Sales' : 'Monthly Expenses'}
          </h2>
          <DataTable 
            columns={reportColumns[reportType] || []} 
            data={getReportData()} 
            keyField={reportType === 'sales' ? 'date' : 'month'}
            actions={[]}
          />
        </div>
      </div>
    </div>
  );
}

// ============================================
// RESERVATIONS PAGE
// ============================================
function ReservationsPage() {
  const [reservations, setReservations] = useState([
    { 
      id: 1, reservationNumber: 'RES-20260727-001', table: 'Table 1', tableId: 1,
      customer: 'John Doe', customerPhone: '+91 9876543210',
      guests: 4, startTime: '2026-07-27T19:00:00', endTime: '2026-07-27T20:30:00',
      status: 'Confirmed', specialRequests: 'Window seat preferred'
    },
    { 
      id: 2, reservationNumber: 'RES-20260727-002', table: 'Table 2', tableId: 2,
      customer: 'Priya Sharma', customerPhone: '+91 9876543211',
      guests: 2, startTime: '2026-07-27T20:00:00', endTime: '2026-07-27T21:30:00',
      status: 'Confirmed', specialRequests: 'Quiet table'
    },
    { 
      id: 3, reservationNumber: 'RES-20260728-001', table: 'Table 3', tableId: 3,
      customer: 'Rahul Gupta', customerPhone: '+91 9876543212',
      guests: 6, startTime: '2026-07-28T20:30:00', endTime: '2026-07-28T22:00:00',
      status: 'Pending', specialRequests: 'Vegan menu'
    },
  ]);
  const [tables] = useState([
    { id: 1, number: 1, name: 'Table 1', capacity: 4, status: 'available' },
    { id: 2, number: 2, name: 'Table 2', capacity: 4, status: 'available' },
    { id: 3, number: 3, name: 'Table 3', capacity: 6, status: 'available' },
    { id: 4, number: 4, name: 'Table 4', capacity: 4, status: 'available' },
  ]);
  const [statusFilter, setStatusFilter] = useState('all');
  const [showReservationModal, setShowReservationModal] = useState(false);
  const [editingReservation, setEditingReservation] = useState(null);

  const statusOptions = ['All', 'Confirmed', 'Pending', 'Cancelled', 'Completed'];

  const filteredReservations = reservations.filter(reservation => {
    const matchesStatus = statusFilter === 'all' || reservation.status.toLowerCase() === statusFilter.toLowerCase();
    return matchesStatus;
  });

  const reservationColumns = [
    { key: 'reservationNumber', label: 'Reservation #' },
    { key: 'customer', label: 'Customer' },
    { key: 'customerPhone', label: 'Phone' },
    { key: 'table', label: 'Table' },
    { key: 'guests', label: 'Guests' },
    { 
      key: 'startTime', 
      label: 'Date & Time',
      render: (value) => new Date(value).toLocaleString('en-IN', { 
        day: '2-digit', month: 'short', year: 'numeric', 
        hour: '2-digit', minute: '2-digit', hour12: false 
      })
    },
    { 
      key: 'status', 
      label: 'Status',
      render: (value) => (
        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
          value === 'Confirmed' ? 'bg-green-100 text-green-700 dark:bg-green-900 dark:text-green-300' :
          value === 'Pending' ? 'bg-orange-100 text-orange-700 dark:bg-orange-900 dark:text-orange-300' :
          value === 'Cancelled' ? 'bg-red-100 text-red-700 dark:bg-red-900 dark:text-red-300' :
          'bg-slate-100 text-slate-700 dark:bg-slate-900 dark:text-slate-300'
        }`}>
          {value}
        </span>
      )
    }
  ];

  const handleReservationSubmit = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const newReservation = {
      id: editingReservation?.id || reservations.length + 1,
      reservationNumber: editingReservation?.reservationNumber || `RES-${new Date().toISOString().split('T')[0].replace(/-/g, '')}-${String(reservations.length + 1).padStart(3, '0')}`,
      table: tables.find(t => t.id.toString() === formData.get('tableId'))?.name || '',
      tableId: parseInt(formData.get('tableId')) || 1,
      customer: formData.get('customer'),
      customerPhone: formData.get('customerPhone'),
      guests: parseInt(formData.get('guests')) || 1,
      startTime: `${formData.get('date')}T${formData.get('startTime')}:00`,
      endTime: `${formData.get('date')}T${formData.get('endTime')}:00`,
      status: formData.get('status') || 'Pending',
      specialRequests: formData.get('specialRequests')
    };
    
    if (editingReservation) {
      setReservations(reservations.map(r => r.id === newReservation.id ? newReservation : r));
    } else {
      setReservations([...reservations, newReservation]);
    }
    
    setShowReservationModal(false);
    setEditingReservation(null);
    alert(`Reservation ${editingReservation ? 'updated' : 'created'} successfully!`);
  };

  const handleDeleteReservation = (reservation) => {
    if (confirm(`Cancel reservation "${reservation.reservationNumber}"?`)) {
      setReservations(reservations.map(r => 
        r.id === reservation.id ? { ...r, status: 'Cancelled' } : r
      ));
    }
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Reservations</h1>
          <p className="text-slate-500 dark:text-slate-400 mt-1">Manage table bookings</p>
        </div>
        <button 
          onClick={() => { setEditingReservation(null); setShowReservationModal(true); }}
          className="flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
        >
          <Plus className="w-4 h-4" />
          <span>New Reservation</span>
        </button>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <StatCard 
          title="Total Reservations" 
          value={reservations.length} 
          icon={Calendar} 
          color="bg-blue-500"
        />
        <StatCard 
          title="Confirmed" 
          value={reservations.filter(r => r.status === 'Confirmed').length} 
          icon={CheckCircle} 
          color="bg-green-500"
        />
        <StatCard 
          title="Pending" 
          value={reservations.filter(r => r.status === 'Pending').length} 
          icon={Clock} 
          color="bg-orange-500"
        />
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <select 
            value={statusFilter}
            onChange={(e) => setStatusFilter(e.target.value)}
            className="px-4 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            {statusOptions.map(option => (
              <option key={option.toLowerCase()} value={option.toLowerCase()}>
                {option}
              </option>
            ))}
          </select>
          <button className="flex items-center justify-center gap-2 px-4 py-2 bg-slate-100 dark:bg-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-200 dark:hover:bg-slate-600 transition-colors">
            <RefreshCw className="w-4 h-4" />
            <span>Refresh</span>
          </button>
        </div>
      </div>

      <DataTable 
        columns={reservationColumns} 
        data={filteredReservations} 
        keyField="id" 
        onView={(reservation) => alert(`View: ${reservation.reservationNumber}`)}
        onEdit={(reservation) => { setEditingReservation(reservation); setShowReservationModal(true); }}
        onDelete={handleDeleteReservation}
        actions={['view', 'edit', 'delete']}
      />

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">Table Availability</h2>
        <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
          {tables.map(table => {
            const reservation = reservations.find(r => r.tableId === table.id && r.status !== 'Cancelled');
            const isAvailable = !reservation;
            
            return (
              <div 
                key={table.id} 
                className={`p-4 rounded-lg border-2 cursor-pointer transition-colors ${
                  isAvailable 
                    ? 'border-green-500 bg-green-50 dark:bg-green-900/20 hover:bg-green-100 dark:hover:bg-green-900/40' 
                    : 'border-red-500 bg-red-50 dark:bg-red-900/20 hover:bg-red-100 dark:hover:bg-red-900/40'
                }`}
              >
                <p className="font-semibold text-slate-800 dark:text-slate-100">{table.name}</p>
                <p className="text-sm text-slate-500 dark:text-slate-400">Capacity: {table.capacity}</p>
                {!isAvailable && reservation && (
                  <div className="mt-2">
                    <p className="text-xs text-red-600 dark:text-red-400">
                      Reserved for {reservation.customer}
                    </p>
                    <p className="text-xs text-slate-500 dark:text-slate-400">
                      {new Date(reservation.startTime).toLocaleTimeString('en-IN', { 
                        hour: '2-digit', minute: '2-digit', hour12: false 
                      })}
                    </p>
                  </div>
                )}
                <p className="text-xs text-slate-500 dark:text-slate-400 mt-1">
                  Status: {isAvailable ? 'Available' : 'Reserved'}
                </p>
              </div>
            );
          })}
        </div>
      </div>

      <Modal 
        title={editingReservation ? 'Edit Reservation' : 'New Reservation'} 
        isOpen={showReservationModal} 
        onClose={() => { setShowReservationModal(false); setEditingReservation(null); }}
        size="lg"
      >
        <form onSubmit={handleReservationSubmit} className="space-y-4">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <FormInput 
              label="Customer Name" 
              type="text" 
              value={editingReservation?.customer || ''} 
              onChange={(e) => setEditingReservation({...editingReservation, customer: e.target.value})}
              placeholder="John Doe"
              required
            />
            <FormInput 
              label="Customer Phone" 
              type="tel" 
              value={editingReservation?.customerPhone || ''} 
              onChange={(e) => setEditingReservation({...editingReservation, customerPhone: e.target.value})}
              placeholder="+91 9876543210"
              required
            />
          </div>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
            <FormInput 
              label="Table" 
              type="select" 
              value={editingReservation?.tableId || ''} 
              onChange={(e) => setEditingReservation({
                ...editingReservation,
                tableId: parseInt(e.target.value),
                table: tables.find(t => t.id.toString() === e.target.value)?.name || ''
              })}
              placeholder="Select table"
              required
              options={tables.map(t => ({ value: t.id, label: `${t.name} (Capacity: ${t.capacity})` }))}
            />
            <FormInput 
              label="Number of Guests" 
              type="number" 
              value={editingReservation?.guests || ''} 
              onChange={(e) => setEditingReservation({...editingReservation, guests: parseInt(e.target.value) || 1})}
              placeholder="4"
              required
            />
            <FormInput 
              label="Status" 
              type="select" 
              value={editingReservation?.status || ''} 
              onChange={(e) => setEditingReservation({...editingReservation, status: e.target.value})}
              placeholder="Select status"
              required
              options={statusOptions.slice(0, -1).map(s => ({ value: s, label: s }))}
            />
          </div>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <FormInput 
              label="Date" 
              type="date" 
              value={editingReservation?.startTime ? new Date(editingReservation.startTime).toISOString().split('T')[0] : ''} 
              onChange={(e) => setEditingReservation({
                ...editingReservation,
                startTime: `${e.target.value}T${editingReservation?.startTime ? new Date(editingReservation.startTime).toISOString().split('T')[1] : '19:00:00'}`
              })}
              required
            />
            <div className="grid grid-cols-2 gap-4">
              <FormInput 
                label="Start Time" 
                type="time" 
                value={editingReservation?.startTime ? new Date(editingReservation.startTime).toISOString().split('T')[1].substring(0, 5) : ''} 
                onChange={(e) => {
                  const date = editingReservation?.startTime ? new Date(editingReservation.startTime).toISOString().split('T')[0] : new Date().toISOString().split('T')[0];
                  setEditingReservation({
                    ...editingReservation,
                    startTime: `${date}T${e.target.value}:00`
                  });
                }}
                required
              />
              <FormInput 
                label="End Time" 
                type="time" 
                value={editingReservation?.endTime ? new Date(editingReservation.endTime).toISOString().split('T')[1].substring(0, 5) : ''} 
                onChange={(e) => {
                  const date = editingReservation?.endTime ? new Date(editingReservation.endTime).toISOString().split('T')[0] : new Date().toISOString().split('T')[0];
                  setEditingReservation({
                    ...editingReservation,
                    endTime: `${date}T${e.target.value}:00`
                  });
                }}
                required
              />
            </div>
          </div>
          <FormInput 
            label="Special Requests" 
            type="textarea" 
            value={editingReservation?.specialRequests || ''} 
            onChange={(e) => setEditingReservation({...editingReservation, specialRequests: e.target.value})}
            placeholder="e.g., Window seat, Vegan menu, etc."
          />
          <div className="flex items-center justify-end gap-2">
            <button 
              type="button"
              onClick={() => { setShowReservationModal(false); setEditingReservation(null); }}
              className="px-4 py-2 border border-slate-200 dark:border-slate-700 text-slate-800 dark:text-slate-100 rounded-lg hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors"
            >
              Cancel
            </button>
            <button 
              type="submit"
              className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors"
            >
              {editingReservation ? 'Update Reservation' : 'Create Reservation'}
            </button>
          </div>
        </form>
      </Modal>
    </div>
  );
}

// ============================================
// SETTINGS PAGE
// ============================================
function SettingsPage() {
  const [settings, setSettings] = useState({
    restaurantName: 'My Restaurant',
    restaurantAddress: '123 Main Street, Mumbai, Maharashtra 400001',
    restaurantPhone: '+91 9876543210',
    restaurantEmail: 'info@myrestaurant.com',
    restaurantWebsite: 'https://myrestaurant.com',
    taxRate: 10,
    serviceCharge: 5,
    currency: '₹',
    timezone: 'Asia/Kolkata',
    openingTime: '09:00',
    closingTime: '22:00',
    maxTableSize: 10,
    defaultReservationDuration: 90,
    loyaltyPointsRate: 10,
    loyaltyRupeeRate: 1
  });
  const [isLoading, setIsLoading] = useState(false);

  const handleSave = (e) => {
    e.preventDefault();
    setIsLoading(true);
    
    setTimeout(() => {
      alert('Settings saved successfully!');
      setIsLoading(false);
    }, 1000);
  };

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Settings</h1>
        <p className="text-slate-500 dark:text-slate-400 mt-1">Configure your restaurant settings</p>
      </div>

      <form onSubmit={handleSave} className="space-y-6">
        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">General Settings</h2>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <FormInput 
              label="Restaurant Name" 
              type="text" 
              value={settings.restaurantName} 
              onChange={(e) => setSettings({...settings, restaurantName: e.target.value})}
              placeholder="My Restaurant"
              required
            />
            <FormInput 
              label="Restaurant Email" 
              type="email" 
              value={settings.restaurantEmail} 
              onChange={(e) => setSettings({...settings, restaurantEmail: e.target.value})}
              placeholder="info@myrestaurant.com"
              required
            />
          </div>
          <FormInput 
            label="Restaurant Address" 
            type="textarea" 
            value={settings.restaurantAddress} 
            onChange={(e) => setSettings({...settings, restaurantAddress: e.target.value})}
            placeholder="123 Main Street, City, State, ZIP"
          />
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <FormInput 
              label="Restaurant Phone" 
              type="tel" 
              value={settings.restaurantPhone} 
              onChange={(e) => setSettings({...settings, restaurantPhone: e.target.value})}
              placeholder="+91 9876543210"
            />
            <FormInput 
              label="Restaurant Website" 
              type="url" 
              value={settings.restaurantWebsite} 
              onChange={(e) => setSettings({...settings, restaurantWebsite: e.target.value})}
              placeholder="https://myrestaurant.com"
            />
          </div>
        </div>

        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">Financial Settings</h2>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
            <FormInput 
              label="Tax Rate (%)" 
              type="number" 
              value={settings.taxRate} 
              onChange={(e) => setSettings({...settings, taxRate: parseFloat(e.target.value) || 0})}
              placeholder="10"
              required
            />
            <FormInput 
              label="Service Charge (%)" 
              type="number" 
              value={settings.serviceCharge} 
              onChange={(e) => setSettings({...settings, serviceCharge: parseFloat(e.target.value) || 0})}
              placeholder="5"
            />
            <FormInput 
              label="Currency Symbol" 
              type="text" 
              value={settings.currency} 
              onChange={(e) => setSettings({...settings, currency: e.target.value})}
              placeholder="₹"
              required
            />
          </div>
        </div>

        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">Operating Hours</h2>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <FormInput 
              label="Opening Time" 
              type="time" 
              value={settings.openingTime} 
              onChange={(e) => setSettings({...settings, openingTime: e.target.value})}
              required
            />
            <FormInput 
              label="Closing Time" 
              type="time" 
              value={settings.closingTime} 
              onChange={(e) => setSettings({...settings, closingTime: e.target.value})}
              required
            />
          </div>
        </div>

        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">Reservation Settings</h2>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <FormInput 
              label="Maximum Table Size" 
              type="number" 
              value={settings.maxTableSize} 
              onChange={(e) => setSettings({...settings, maxTableSize: parseInt(e.target.value) || 0})}
              placeholder="10"
              required
            />
            <FormInput 
              label="Default Reservation Duration (minutes)" 
              type="number" 
              value={settings.defaultReservationDuration} 
              onChange={(e) => setSettings({...settings, defaultReservationDuration: parseInt(e.target.value) || 0})}
              placeholder="90"
              required
            />
          </div>
        </div>

        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">Loyalty Program Settings</h2>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <FormInput 
              label="Points per ₹100 Spent" 
              type="number" 
              value={settings.loyaltyPointsRate} 
              onChange={(e) => setSettings({...settings, loyaltyPointsRate: parseInt(e.target.value) || 0})}
              placeholder="10"
              required
            />
            <FormInput 
              label="₹1 = ? Points" 
              type="number" 
              value={settings.loyaltyRupeeRate} 
              onChange={(e) => setSettings({...settings, loyaltyRupeeRate: parseInt(e.target.value) || 0})}
              placeholder="1"
              required
            />
          </div>
        </div>

        <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
          <h2 className="text-lg font-semibold text-slate-800 dark:text-slate-100 mb-4">Regional Settings</h2>
          <FormInput 
            label="Timezone" 
            type="select" 
            value={settings.timezone} 
            onChange={(e) => setSettings({...settings, timezone: e.target.value})}
            required
            options={[
              { value: 'Asia/Kolkata', label: 'Asia/Kolkata (IST, UTC+5:30)' },
              { value: 'UTC', label: 'UTC (Coordinated Universal Time)' },
              { value: 'America/New_York', label: 'America/New_York (EST, UTC-5)' },
            ]}
          />
        </div>

        <div className="flex items-center justify-end">
          <button 
            type="submit" 
            disabled={isLoading}
            className="flex items-center gap-2 px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {isLoading ? (
              <>
                <div className="w-5 h-5 border-2 border-white border-t-transparent rounded-full animate-spin"></div>
                <span>Saving...</span>
              </>
            ) : (
              <>
                <Settings className="w-5 h-5" />
                <span>Save All Settings</span>
              </>
            )}
          </button>
        </div>
      </form>
    </div>
  );
}

// ============================================
// LOGIN PAGE
// ============================================
function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [rememberMe, setRememberMe] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const { login } = useContext(AuthContext);
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsLoading(true);
    
    try {
      await new Promise(resolve => setTimeout(resolve, 1000));
      await login(email, password);
      navigate('/');
    } catch (error) {
      alert('Invalid credentials. Please try again.');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-slate-900 to-slate-700 p-4">
      <div className="bg-white dark:bg-slate-800 rounded-2xl p-8 w-full max-w-md shadow-2xl">
        <div className="text-center mb-8">
          <div className="w-16 h-16 bg-gradient-to-br from-blue-500 to-purple-600 rounded-2xl flex items-center justify-center mx-auto mb-4">
            <Restaurant className="w-8 h-8 text-white" />
          </div>
          <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Restaurant Management System</h1>
          <p className="text-slate-500 dark:text-slate-400 mt-2">Sign in to your account</p>
        </div>

        <form onSubmit={handleSubmit} className="space-y-5">
          <div>
            <label className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">
              Email Address
            </label>
            <input 
              type="email" 
              value={email}
              onChange={(e) => setEmail(e.target.value)}
              placeholder="Enter your email"
              required
              className="w-full px-4 py-3 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
          <div>
            <label className="block text-sm font-medium text-slate-700 dark:text-slate-300 mb-1">
              Password
            </label>
            <input 
              type="password" 
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              placeholder="Enter your password"
              required
              className="w-full px-4 py-3 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
          <div className="flex items-center justify-between">
            <label className="flex items-center gap-2 cursor-pointer">
              <input 
                type="checkbox" 
                checked={rememberMe}
                onChange={(e) => setRememberMe(e.target.checked)}
                className="w-4 h-4 rounded border-slate-200 dark:border-slate-700 text-blue-600 focus:ring-blue-500"
              />
              <span className="text-sm text-slate-700 dark:text-slate-300">Remember me</span>
            </label>
            <a href="#" className="text-sm text-blue-600 hover:text-blue-700">
              Forgot password?
            </a>
          </div>
          <button 
            type="submit" 
            disabled={isLoading}
            className="w-full py-3 bg-blue-600 text-white rounded-lg font-medium hover:bg-blue-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center gap-2"
          >
            {isLoading ? (
              <div className="w-5 h-5 border-2 border-white border-t-transparent rounded-full animate-spin"></div>
            ) : (
              <>Sign In <User className="w-5 h-5" /></>
            )}
          </button>
        </form>

        <div className="mt-6 text-center space-y-2">
          <p className="text-sm text-slate-500 dark:text-slate-400">
            Don't have an account?{' '}
            <Link to="/contact-admin" className="text-blue-600 hover:text-blue-700 font-medium">
              Contact Admin
            </Link>
          </p>
          <p className="text-sm text-slate-500 dark:text-slate-400">
            <Link to="/customer-dashboard" className="text-blue-600 hover:text-blue-700 font-medium">
              Customer Portal →
            </Link>
          </p>
        </div>
      </div>
    </div>
  );
}

// ============================================
// PROTECTED ROUTE COMPONENT
// ============================================
function ProtectedRoute({ children }) {
  const { isAuthenticated } = useContext(AuthContext);
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

// ============================================
// MAIN APP COMPONENT
// ============================================
export default function App() {
  return (
    <ThemeProvider>
      <AuthProvider>
        <Router>
          <Routes>
            <Route path="/login" element={<LoginPage />} />
            <Route 
              path="/" 
              element={
                <ProtectedRoute>
                  <Layout><DashboardPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/orders" 
              element={
                <ProtectedRoute>
                  <Layout><OrdersPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/menu" 
              element={
                <ProtectedRoute>
                  <Layout><MenuPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/inventory" 
              element={
                <ProtectedRoute>
                  <Layout><InventoryPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/billing" 
              element={
                <ProtectedRoute>
                  <Layout><BillingPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/customers" 
              element={
                <ProtectedRoute>
                  <Layout><CustomersPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/staff" 
              element={
                <ProtectedRoute>
                  <Layout><StaffPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/reports" 
              element={
                <ProtectedRoute>
                  <Layout><ReportsPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/reservations" 
              element={
                <ProtectedRoute>
                  <Layout><ReservationsPage /></Layout>
                </ProtectedRoute>
              }
            />
            <Route 
              path="/settings" 
              element={
                <ProtectedRoute>
                  <Layout><SettingsPage /></Layout>
                </ProtectedRoute>
              }
            />
            {/* Public Routes (No Auth Required) */}
            <Route path="/customer-dashboard" element={<PublicLayout><CustomerDashboardPage /></PublicLayout>} />
            <Route path="/contact-admin" element={<PublicLayout><ContactAdminPage /></PublicLayout>} />
            <Route path="*" element={<Navigate to="/" replace />} />
          </Routes>
        </Router>
      </AuthProvider>
    </ThemeProvider>
  );
}

// ============================================
// CONTACT ADMIN PAGE
// ============================================
function ContactAdminPage() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    subject: '',
    message: ''
  });
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitStatus, setSubmitStatus] = useState(null);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 1500));
    
    setIsSubmitting(false);
    setSubmitStatus('success');
    setFormData({ name: '', email: '', subject: '', message: '' });
    
    // Reset status after 5 seconds
    setTimeout(() => setSubmitStatus(null), 5000);
  };

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Contact Admin</h1>
      </div>

      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700 max-w-2xl">
        <h2 className="text-lg font-semibold text-slate-700 dark:text-slate-200 mb-4">Send a Message</h2>
        
        {submitStatus === 'success' ? (
          <div className="bg-green-50 dark:bg-green-900/20 border border-green-200 dark:border-green-800 text-green-800 dark:text-green-400 p-4 rounded-lg mb-4">
            Your message has been sent successfully! We will get back to you soon.
          </div>
        ) : null}

        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-slate-600 dark:text-slate-300 mb-1">Name</label>
            <input
              type="text"
              name="name"
              value={formData.name}
              onChange={handleChange}
              required
              className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Your name"
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-slate-600 dark:text-slate-300 mb-1">Email</label>
            <input
              type="email"
              name="email"
              value={formData.email}
              onChange={handleChange}
              required
              className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Your email"
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-slate-600 dark:text-slate-300 mb-1">Subject</label>
            <input
              type="text"
              name="subject"
              value={formData.subject}
              onChange={handleChange}
              required
              className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Subject"
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-slate-600 dark:text-slate-300 mb-1">Message</label>
            <textarea
              name="message"
              value={formData.message}
              onChange={handleChange}
              required
              rows={5}
              className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="Your message"
            />
          </div>

          <button
            type="submit"
            disabled={isSubmitting}
            className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
          >
            {isSubmitting ? 'Sending...' : 'Send Message'}
          </button>
        </form>

        <div className="mt-6 p-4 bg-slate-50 dark:bg-slate-700 rounded-lg">
          <h3 className="font-medium text-slate-700 dark:text-slate-200 mb-2">Contact Information</h3>
          <p className="text-sm text-slate-600 dark:text-slate-400">Email: admin@restaurant.com</p>
          <p className="text-sm text-slate-600 dark:text-slate-400">Phone: +1 (123) 456-7890</p>
          <p className="text-sm text-slate-600 dark:text-slate-400">Address: 123 Food Street, City, Country</p>
        </div>
      </div>
    </div>
  );
}

// ============================================
// CUSTOMER DASHBOARD PAGE
// ============================================
function CustomerDashboardPage() {
  useEffect(() => {
    document.title = 'Customer Dashboard - RMS';
  }, []);
  
  const [activeTab, setActiveTab] = useState('book');
  const [bookings, setBookings] = useState([
    { id: 1, date: '2026-08-01', time: '7:00 PM', guests: 4, table: 'Table 5', status: 'confirmed' },
    { id: 2, date: '2026-08-05', time: '8:30 PM', guests: 2, table: 'Table 12', status: 'pending' }
  ]);
  const [orders, setOrders] = useState([
    { id: 1, orderId: '#ORD-001', date: '2026-07-25', items: 3, total: 85.50, status: 'delivered' },
    { id: 2, orderId: '#ORD-002', date: '2026-07-20', items: 5, total: 120.75, status: 'delivered' }
  ]);
  const [cart, setCart] = useState([]);
  const [menuItems, setMenuItems] = useState([
    { id: 1, name: 'Margherita Pizza', price: 12.99, category: 'Pizza', description: 'Classic tomato and mozzarella' },
    { id: 2, name: 'Pasta Carbonara', price: 14.99, category: 'Pasta', description: 'Creamy pasta with bacon and parmesan' },
    { id: 3, name: 'Caesar Salad', price: 8.99, category: 'Salad', description: 'Fresh romaine with caesar dressing' },
    { id: 4, name: 'Chicken Burger', price: 10.99, category: 'Burger', description: 'Grilled chicken with fresh veggies' },
    { id: 5, name: 'Chocolate Lava Cake', price: 6.99, category: 'Dessert', description: 'Warm chocolate cake with vanilla ice cream' },
    { id: 6, name: 'Mushroom Risotto', price: 13.99, category: 'Rice', description: 'Creamy risotto with wild mushrooms' }
  ]);

  const addToCart = (item) => {
    setCart(prev => {
      const existingItem = prev.find(i => i.id === item.id);
      if (existingItem) {
        return prev.map(i => i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i);
      }
      return [...prev, { ...item, quantity: 1 }];
    });
  };

  const removeFromCart = (itemId) => {
    setCart(prev => prev.filter(item => item.id !== itemId));
  };

  const updateQuantity = (itemId, newQuantity) => {
    if (newQuantity <= 0) {
      removeFromCart(itemId);
      return;
    }
    setCart(prev => prev.map(item => item.id === itemId ? { ...item, quantity: newQuantity } : item));
  };

  const placeOrder = () => {
    if (cart.length === 0) return;
    
    const newOrder = {
      id: Date.now(),
      orderId: `#ORD-${Date.now().toString().slice(-4)}`,
      date: new Date().toISOString().split('T')[0],
      items: cart.length,
      total: cart.reduce((sum, item) => sum + (item.price * item.quantity), 0),
      status: 'processing'
    };
    
    setOrders(prev => [newOrder, ...prev]);
    setCart([]);
    alert('Order placed successfully!');
  };

  const cartTotal = cart.reduce((sum, item) => sum + (item.price * item.quantity), 0);

  // Booking form state
  const [bookingForm, setBookingForm] = useState({
    date: '',
    time: '',
    guests: 1,
    specialRequest: ''
  });

  const handleBookingChange = (e) => {
    const { name, value } = e.target;
    setBookingForm(prev => ({ ...prev, [name]: value }));
  };

  const makeBooking = () => {
    if (!bookingForm.date || !bookingForm.time) return;
    
    const newBooking = {
      id: Date.now(),
      date: bookingForm.date,
      time: bookingForm.time,
      guests: bookingForm.guests,
      table: `Table ${Math.floor(Math.random() * 20) + 1}`,
      status: 'pending'
    };
    
    setBookings(prev => [newBooking, ...prev]);
    setBookingForm({ date: '', time: '', guests: 1, specialRequest: '' });
    alert('Booking request sent! We will confirm shortly.');
  };

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h1 className="text-2xl font-bold text-slate-800 dark:text-slate-100">Customer Dashboard</h1>
        <div className="flex gap-2">
          <Link to="/contact-admin" className="px-4 py-2 bg-slate-200 dark:bg-slate-700 text-slate-800 dark:text-slate-200 rounded-lg hover:bg-slate-300 dark:hover:bg-slate-600 transition-colors">
            Contact Admin
          </Link>
        </div>
      </div>

      {/* Customer Stats */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <StatCard
          title="Active Bookings"
          value={bookings.filter(b => b.status === 'confirmed').length}
          icon={Calendar}
          color="bg-blue-500"
        />
        <StatCard
          title="Total Orders"
          value={orders.length}
          icon={Receipt}
          color="bg-green-500"
        />
        <StatCard
          title="Cart Total"
          value={`$${cartTotal.toFixed(2)}`}
          icon={ShoppingCart}
          color="bg-orange-500"
        />
      </div>

      {/* Tabs */}
      <div className="bg-white dark:bg-slate-800 rounded-xl p-6 shadow-sm border border-slate-200 dark:border-slate-700">
        <div className="flex border-b border-slate-200 dark:border-slate-700 mb-6">
          <button
            onClick={() => setActiveTab('book')}
            className={`px-4 py-2 font-medium text-sm transition-colors ${activeTab === 'book' ? 'border-b-2 border-blue-500 text-blue-600 dark:text-blue-400' : 'text-slate-500 hover:text-slate-700 dark:text-slate-400'}`}
          >
            Book a Table
          </button>
          <button
            onClick={() => setActiveTab('order')}
            className={`px-4 py-2 font-medium text-sm transition-colors ${activeTab === 'order' ? 'border-b-2 border-blue-500 text-blue-600 dark:text-blue-400' : 'text-slate-500 hover:text-slate-700 dark:text-slate-400'}`}
          >
            Order Online
          </button>
          <button
            onClick={() => setActiveTab('bookings')}
            className={`px-4 py-2 font-medium text-sm transition-colors ${activeTab === 'bookings' ? 'border-b-2 border-blue-500 text-blue-600 dark:text-blue-400' : 'text-slate-500 hover:text-slate-700 dark:text-slate-400'}`}
          >
            My Bookings
          </button>
          <button
            onClick={() => setActiveTab('orders')}
            className={`px-4 py-2 font-medium text-sm transition-colors ${activeTab === 'orders' ? 'border-b-2 border-blue-500 text-blue-600 dark:text-blue-400' : 'text-slate-500 hover:text-slate-700 dark:text-slate-400'}`}
          >
            Order History
          </button>
        </div>

        {/* Tab Content */}
        <div className="space-y-6">
          {activeTab === 'book' && (
            <div className="space-y-4">
              <h2 className="text-lg font-semibold text-slate-700 dark:text-slate-200">Book a Table</h2>
              
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label className="block text-sm font-medium text-slate-600 dark:text-slate-300 mb-1">Date</label>
                  <input
                    type="date"
                    name="date"
                    value={bookingForm.date}
                    onChange={handleBookingChange}
                    required
                    className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-600 dark:text-slate-300 mb-1">Time</label>
                  <select
                    name="time"
                    value={bookingForm.time}
                    onChange={handleBookingChange}
                    required
                    className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100"
                  >
                    <option value="">Select Time</option>
                    <option value="6:00 PM">6:00 PM</option>
                    <option value="7:00 PM">7:00 PM</option>
                    <option value="8:00 PM">8:00 PM</option>
                    <option value="9:00 PM">9:00 PM</option>
                    <option value="10:00 PM">10:00 PM</option>
                  </select>
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-600 dark:text-slate-300 mb-1">Number of Guests</label>
                  <input
                    type="number"
                    name="guests"
                    value={bookingForm.guests}
                    onChange={handleBookingChange}
                    min={1}
                    max={20}
                    className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100"
                  />
                </div>
                <div>
                  <label className="block text-sm font-medium text-slate-600 dark:text-slate-300 mb-1">Special Requests</label>
                  <textarea
                    name="specialRequest"
                    value={bookingForm.specialRequest}
                    onChange={handleBookingChange}
                    placeholder="Any special requests?"
                    className="w-full px-3 py-2 border border-slate-200 dark:border-slate-700 rounded-lg bg-slate-50 dark:bg-slate-700 text-slate-800 dark:text-slate-100"
                  />
                </div>
              </div>
              
              <button
                onClick={makeBooking}
                className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg transition-colors"
              >
                Submit Booking Request
              </button>
            </div>
          )}

          {activeTab === 'order' && (
            <div className="space-y-4">
              <h2 className="text-lg font-semibold text-slate-700 dark:text-slate-200">Order Online</h2>
              
              <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                {menuItems.map(item => (
                  <div key={item.id} className="bg-slate-50 dark:bg-slate-700 rounded-lg p-4 border border-slate-200 dark:border-slate-600">
                    <div className="flex justify-between items-start">
                      <div>
                        <h3 className="font-medium text-slate-800 dark:text-slate-100">{item.name}</h3>
                        <p className="text-sm text-slate-500 dark:text-slate-400">{item.category}</p>
                        <p className="text-sm text-slate-600 dark:text-slate-300 mt-1">{item.description}</p>
                        <p className="text-lg font-bold text-slate-800 dark:text-slate-100 mt-2">${item.price.toFixed(2)}</p>
                      </div>
                      <button
                        onClick={() => addToCart(item)}
                        className="px-3 py-1 bg-blue-600 hover:bg-blue-700 text-white text-sm rounded-lg transition-colors"
                      >
                        Add to Cart
                      </button>
                    </div>
                  </div>
                ))}
              </div>

              {/* Cart Sidebar */}
              {cart.length > 0 && (
                <div className="fixed top-16 right-6 w-80 bg-white dark:bg-slate-800 rounded-xl p-6 shadow-lg border border-slate-200 dark:border-slate-700">
                  <h3 className="font-semibold text-slate-800 dark:text-slate-100 mb-4">Your Cart ({cart.length} items)</h3>
                  
                  <div className="space-y-3 max-h-60 overflow-y-auto">
                    {cart.map(item => (
                      <div key={item.id} className="flex justify-between items-center p-2 bg-slate-50 dark:bg-slate-700 rounded-lg">
                        <div>
                          <p className="font-medium text-slate-800 dark:text-slate-100">{item.name}</p>
                          <p className="text-sm text-slate-500 dark:text-slate-400">${item.price} x {item.quantity}</p>
                        </div>
                        <div className="flex items-center gap-2">
                          <input
                            type="number"
                            value={item.quantity}
                            onChange={(e) => updateQuantity(item.id, parseInt(e.target.value))}
                            min={1}
                            className="w-12 text-center text-sm border border-slate-200 dark:border-slate-600 rounded bg-slate-50 dark:bg-slate-600"
                          />
                          <button
                            onClick={() => removeFromCart(item.id)}
                            className="text-red-500 hover:text-red-700"
                          >
                            <Trash2 className="w-4 h-4" />
                          </button>
                        </div>
                      </div>
                    ))}
                  </div>

                  <div className="mt-4 pt-4 border-t border-slate-200 dark:border-slate-700">
                    <div className="flex justify-between font-medium text-slate-800 dark:text-slate-100">
                      <span>Total:</span>
                      <span>${cartTotal.toFixed(2)}</span>
                    </div>
                    <button
                      onClick={placeOrder}
                      className="w-full mt-3 px-4 py-2 bg-green-600 hover:bg-green-700 text-white rounded-lg transition-colors"
                    >
                      Place Order
                    </button>
                  </div>
                </div>
              )}
            </div>
          )}

          {activeTab === 'bookings' && (
            <div className="space-y-4">
              <h2 className="text-lg font-semibold text-slate-700 dark:text-slate-200">My Bookings</h2>
              
              {bookings.length === 0 ? (
                <p className="text-slate-500 dark:text-slate-400">No bookings found.</p>
              ) : (
                <div className="space-y-4">
                  {bookings.map(booking => (
                    <div key={booking.id} className="bg-slate-50 dark:bg-slate-700 rounded-lg p-4 border border-slate-200 dark:border-slate-600">
                      <div className="grid grid-cols-2 md:grid-cols-4 gap-2">
                        <div>
                          <p className="text-sm text-slate-500 dark:text-slate-400">Date</p>
                          <p className="font-medium text-slate-800 dark:text-slate-100">{booking.date}</p>
                        </div>
                        <div>
                          <p className="text-sm text-slate-500 dark:text-slate-400">Time</p>
                          <p className="font-medium text-slate-800 dark:text-slate-100">{booking.time}</p>
                        </div>
                        <div>
                          <p className="text-sm text-slate-500 dark:text-slate-400">Guests</p>
                          <p className="font-medium text-slate-800 dark:text-slate-100">{booking.guests}</p>
                        </div>
                        <div>
                          <p className="text-sm text-slate-500 dark:text-slate-400">Status</p>
                          <span className={`px-2 py-1 rounded-full text-xs font-medium ${
                            booking.status === 'confirmed' ? 'bg-green-100 text-green-800 dark:bg-green-900/20 dark:text-green-400' :
                            booking.status === 'pending' ? 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900/20 dark:text-yellow-400' :
                            'bg-slate-100 text-slate-800 dark:bg-slate-700 dark:text-slate-300'
                          }`}>
                            {booking.status}
                          </span>
                        </div>
                      </div>
                      <div className="mt-2">
                        <p className="text-sm text-slate-500 dark:text-slate-400">Table</p>
                        <p className="font-medium text-slate-800 dark:text-slate-100">{booking.table}</p>
                      </div>
                    </div>
                  ))}
                </div>
              )}
            </div>
          )}

          {activeTab === 'orders' && (
            <div className="space-y-4">
              <h2 className="text-lg font-semibold text-slate-700 dark:text-slate-200">Order History</h2>
              
              {orders.length === 0 ? (
                <p className="text-slate-500 dark:text-slate-400">No orders found.</p>
              ) : (
                <div className="space-y-4">
                  {orders.map(order => (
                    <div key={order.id} className="bg-slate-50 dark:bg-slate-700 rounded-lg p-4 border border-slate-200 dark:border-slate-600">
                      <div className="flex justify-between items-start">
                        <div>
                          <p className="font-medium text-slate-800 dark:text-slate-100">{order.orderId}</p>
                          <p className="text-sm text-slate-500 dark:text-slate-400">{order.date}</p>
                          <p className="text-sm text-slate-600 dark:text-slate-300 mt-1">{order.items} items - ${order.total.toFixed(2)}</p>
                        </div>
                        <span className={`px-2 py-1 rounded-full text-xs font-medium ${
                          order.status === 'delivered' ? 'bg-green-100 text-green-800 dark:bg-green-900/20 dark:text-green-400' :
                          order.status === 'processing' ? 'bg-blue-100 text-blue-800 dark:bg-blue-900/20 dark:text-blue-400' :
                          'bg-slate-100 text-slate-800 dark:bg-slate-700 dark:text-slate-300'
                        }`}>
                          {order.status}
                        </span>
                      </div>
                    </div>
                  ))}
                </div>
              )}
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

// ============================================
// PUBLIC LAYOUT (For Customer Pages)
// ============================================
function PublicLayout({ children }) {
  const { theme } = useContext(ThemeContext);
  
  return (
    <div className={`min-h-screen bg-slate-50 dark:bg-slate-900 ${theme}`}>
      <header className="fixed top-0 left-0 right-0 h-16 bg-white dark:bg-slate-800 border-b border-slate-200 dark:border-slate-700 z-40">
        <div className="h-full px-6 flex items-center justify-between">
          <div className="flex items-center gap-2">
            <Restaurant className="w-6 h-6 text-blue-400" />
            <span className="font-bold text-lg text-slate-800 dark:text-slate-100">RMS - Customer Portal</span>
          </div>
          <div className="flex items-center gap-4">
            <Link to="/" className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg transition-colors">
              Admin Login
            </Link>
          </div>
        </div>
      </header>
      <main className="pt-16 p-6">
        {children}
      </main>
    </div>
  );
}
