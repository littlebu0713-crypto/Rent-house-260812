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
