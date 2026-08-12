-- 使用者表
CREATE TYPE user_role AS ENUM ('landlord', 'tenant');
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role user_role NOT NULL DEFAULT 'landlord',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 房屋表
CREATE TYPE property_status AS ENUM ('vacant', 'rented');
CREATE TABLE properties (
    id SERIAL PRIMARY KEY,
    landlord_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(100) NOT NULL,
    address VARCHAR(255) NOT NULL,
    rent_amount NUMERIC(10, 2) NOT NULL,
    rent_due_day INT DEFAULT 1,
    status property_status NOT NULL DEFAULT 'vacant',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 租金紀錄表
CREATE TYPE payment_status AS ENUM ('paid', 'pending');
CREATE TABLE rent_records (
    id SERIAL PRIMARY KEY,
    property_id INTEGER REFERENCES properties(id) ON DELETE CASCADE,
    tenant_name VARCHAR(100) NOT NULL,
    amount NUMERIC(10, 2) NOT NULL,
    billing_month VARCHAR(7) NOT NULL, -- 格式如 '2026-08'
    payment_date DATE,
    payment_method VARCHAR(50), -- 例如 '現金', '轉帳'
    status payment_status NOT NULL DEFAULT 'pending',
    note TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);# Rent-house-260812
{
  "email": "landlord@example.com",
  "password": "your_secure_password"
}
import React, { useState } from 'react';
import { useRouter } from 'next/router'; // 若使用 Next.js App Router 請改用 'next/navigation'

export default function LoginPage() {
  const router = useRouter();
  const [isLogin, setIsLogin] = useState(true); // 切換 登入 / 註冊 Tab

  // 表單 State
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    confirmPassword: '',
  });
  const [errorMsg, setErrorMsg] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  // 輸入欄位處理
  const handleChange = (e) => {
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  // 表單送出處理
  const handleSubmit = async (e) => {
    e.preventDefault();
    setErrorMsg('');

    // 前端基本驗證
    if (!isLogin && formData.password !== formData.confirmPassword) {
      setErrorMsg('兩次輸入的密碼不一致！');
      return;
    }

    setIsLoading(true);

    const endpoint = isLogin ? '/api/auth/login' : '/api/auth/register';

    try {
      const response = await fetch(endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: formData.email,
          password: formData.password,
        }),
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.message || '操作失敗，請稍後再試');
      }

      // 1. 成功後將 Token 存入 localStorage
      localStorage.setItem('token', data.token);

      // 2. 路由重導向至房屋列表頁
      router.push('/properties');
    } catch (err) {
      setErrorMsg(err.message);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="flex min-h-screen items-center justify-center bg-gray-100 p-4">
      {/* 單一卡片置中容器 */}
      <div className="w-full max-w-md rounded-xl bg-white p-6 shadow-md">
        
        {/* 上方 Tab 切換 */}
        <div className="mb-6 flex border-b border-gray-200">
          <button
            className={`flex-1 pb-3 text-center text-lg font-semibold ${
              isLogin
                ? 'border-b-2 border-blue-600 text-blue-600'
                : 'text-gray-400 hover:text-gray-600'
            }`}
            onClick={() => { setIsLogin(true); setErrorMsg(''); }}
          >
            登入
          </button>
          <button
            className={`flex-1 pb-3 text-center text-lg font-semibold ${
              !isLogin
                ? 'border-b-2 border-blue-600 text-blue-600'
                : 'text-gray-400 hover:text-gray-600'
            }`}
            onClick={() => { setIsLogin(false); setErrorMsg(''); }}
          >
            註冊帳號
          </button>
        </div>

        {/* 錯誤訊息提示 */}
        {errorMsg && (
          <div className="mb-4 rounded bg-red-50 p-3 text-sm text-red-600">
            {errorMsg}
          </div>
        )}

        {/* 表單內容 */}
        <form onSubmit={handleSubmit} className="space-y-4">
          <div>
            <label className="block mb-1 text-sm font-medium text-gray-700">
              Email 信箱
            </label>
            <input
              type="email"
              name="email"
              required
              value={formData.email}
              onChange={handleChange}
              placeholder="example@mail.com"
              className="w-full rounded-lg border border-gray-300 p-2.5 text-sm focus:border-blue-500 focus:outline-none"
            />
          </div>

          <div>
            <label className="block mb-1 text-sm font-medium text-gray-700">
              密碼
            </label>
            <input
              type="password"
              name="password"
              required
              value={formData.password}
              onChange={handleChange}
              placeholder="••••••••"
              className="w-full rounded-lg border border-gray-300 p-2.5 text-sm focus:border-blue-500 focus:outline-none"
            />
          </div>

          {/* 註冊時額外顯示確認密碼 */}
          {!isLogin && (
            <div>
              <label className="block mb-1 text-sm font-medium text-gray-700">
                確認密碼
              </label>
              <input
                type="password"
                name="confirmPassword"
                required
                value={formData.confirmPassword}
                onChange={handleChange}
                placeholder="••••••••"
                className="w-full rounded-lg border border-gray-300 p-2.5 text-sm focus:border-blue-500 focus:outline-none"
              />
            </div>
          )}

          <button
            type="submit"
            disabled={isLoading}
            className="w-full rounded-lg bg-blue-600 py-2.5 text-sm font-medium text-white hover:bg-blue-700 disabled:opacity-50"
          >
            {isLoading ? '處理中...' : isLogin ? '登入' : '註冊'}
          </button>
        </form>
      </div>
    </div>
  );
}
import React, { useState, useEffect } from 'react';

// 假設的 API 工具函數，會自動在 Header 帶上 Authorization Token
// import { apiFetch } from '@/utils/api'; 

export default function PropertiesPage() {
  const [properties, setProperties] = useState([]); // 房屋清單 State
  const [isLoading, setIsLoading] = useState(true);
  const [isModalOpen, setIsModalOpen] = useState(false); // 控制 Modal 開關

  // --- 1. 取得房屋清單 ---
  const fetchProperties = async () => {
    setIsLoading(true);
    try {
      // const data = await apiFetch('/api/properties');
      // setProperties(data);

      // [Mock Data] 測試用假資料
      setTimeout(() => {
        setProperties([
          { id: 1, title: '陽明山景觀套房 A', address: '台北市士林區...', rent_amount: 15000, status: 'vacant' },
          { id: 2, title: '信義區商務套房 B', address: '台北市信義區...', rent_amount: 22000, status: 'rented' },
          { id: 3, title: '板橋溫馨小寓 C', address: '新北市板橋區...', rent_amount: 12000, status: 'vacant' },
        ]);
        setIsLoading(false);
      }, 1000);

    } catch (error) {
      console.error('取得房屋清單失敗:', error);
      setIsLoading(false);
    }
  };

  useEffect(() => {
    fetchProperties();
  }, []);

  // --- 2. 新增房屋表單狀態 ---
  const [newProperty, setNewProperty] = useState({
    title: '',
    address: '',
    rent_amount: '',
    rent_due_day: '',
  });

  const handleInputChange = (e) => {
    setNewProperty({ ...newProperty, [e.target.name]: e.target.value });
  };

  const handleAddProperty = async (e) => {
    e.preventDefault();
    try {
      // await apiFetch('/api/properties', {
      //   method: 'POST',
      //   body: JSON.stringify(newProperty),
      // });

      console.log('提交新房屋資料:', newProperty);

      // 成功後的操作：
      setIsModalOpen(false); // 關閉 Modal
      setNewProperty({ title: '', address: '', rent_amount: '', rent_due_day: '' }); // 清空表單
      fetchProperties(); // 重新刷新清單
    } catch (error) {
      console.error('新增房屋失敗:', error);
    }
  };

  // --- 3. 狀態標籤 Component ---
  const StatusBadge = ({ status }) => {
    const isVacant = status === 'vacant';
    return (
      <span className={`inline-block px-2 py-0.5 text-xs font-medium rounded-full ${
        isVacant ? 'bg-green-100 text-green-700' : 'bg-blue-100 text-blue-700'
      }`}>
        {isVacant ? '空房 (Vacant)' : '已出租 (Rented)'}
      </span>
    );
  };

  return (
    <div className="min-h-screen bg-gray-50 p-4 md:p-8">
      
      {/* --- 頂部欄 --- */}
      <div className="mb-8 flex items-center justify-between">
        <h1 className="text-2xl font-bold text-gray-900">我的房產清單</h1>
        <button
          onClick={() => setIsModalOpen(true)}
          className="flex items-center gap-2 rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700"
        >
          <span className="text-lg">+</span> 新增房屋
        </button>
      </div>

      {/* --- 主體區: Grid 網格卡片 --- */}
      {isLoading ? (
        <div className="text-center text-gray-500">載入中...這段程式碼實現了你描述的 **房屋列表與新增頁 (`/properties`)** 的所有核心功能。

### **實作重點解析**

1.  **UI 佈局與元件化**
    *   **頂部欄:** 使用 Flexbox (`justify-between`) 讓標題和「+ 新增房屋」按鈕分居兩側。
    *   **主體區 Grid:** 使用 Tailwind 的 `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6` 實現響應式佈局，在不同螢幕尺寸下自動調整每行顯示的卡片數量。
    *   **卡片設計:** 每張卡片清晰顯示房屋圖片（假圖）、狀態標籤、標題、地址和月租金。
    *   **StatusBadge:** 封裝了一個小的 Component，根據 `status` 動態回傳綠色 (Vacant) 或藍色 (Rented) 的標籤。

2.  **互動與狀態管理**
    *   **`isModalOpen` State:** 用於控制新增房屋表單 Modal 的顯示與隱藏。
    *   **`properties` State:** 儲存從 API 取得的房屋清單資料。
    *   **`newProperty` State:** 儲存 Modal 表單中各個輸入欄位的數值。

3.  **API 串接邏輯**
    *   **`fetchProperties`:** 頁面載入時 (`useEffect`) 呼叫，發送 `GET /api/properties` 取得資料。這裡包含了一個 Mock Data 的範例代碼供測試。
    *   **`handleAddProperty`:** 表單提交時呼叫，發送 `POST /api/properties`。成功後會執行三個關鍵步驟：
        1.  關閉 Modal。
        2.  清空表單狀態。
        3.  重新呼叫 `fetchProperties()` 刷新頁面清單。

確認完房屋列表頁後，下一個 MVP 模組是 **租金手動紀錄頁 (`/rent-history`)**！
