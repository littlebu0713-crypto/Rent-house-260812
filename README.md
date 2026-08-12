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
