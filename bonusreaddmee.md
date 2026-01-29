# FlowPay - Complete Feature Implementation Guide

> **Every Feature, Every Integration, Every Detail**

---

## 📋 Table of Contents

1. [Authentication & User Management](#1-authentication--user-management)
2. [Digital Wallet](#2-digital-wallet)
3. [P2P Transfers](#3-p2p-transfers)
4. [Bill Payments](#4-bill-payments)
5. [Loans & EMI](#5-loans--emi)
6. [Virtual Cards](#6-virtual-cards)
7. [Merchant & QR Payments](#7-merchant--qr-payments)
8. [Rewards & Cashback](#8-rewards--cashback)
9. [Stock Trading Platform](#9-stock-trading-platform)
10. [Statement Generation](#10-statement-generation)
11. [OTP System](#11-otp-system)
12. [Frontend Features](#12-frontend-features)
13. [Security Features](#13-security-features)
14. [External Integrations](#14-external-integrations)

---

## 1. Authentication & User Management

### 1.1 User Registration

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Email/Password Registration | ✅ Implemented | `auth_service.go` |
| Username uniqueness check | ✅ Implemented | `auth_service.go:59` |
| Email uniqueness check | ✅ Implemented | `auth_service.go:54` |
| Password hashing (bcrypt) | ✅ Implemented | `auth_service.go:64` |
| Automatic wallet creation | ✅ Implemented | `auth_service.go:84` |
| Personal QR code generation | ✅ Implemented | `auth_service.go:236` |
| JWT token generation | ✅ Implemented | `auth_service.go:134` |

**Request Structure:**
```json
{
    "email": "user@example.com",
    "password": "minimum8chars",
    "full_name": "John Doe",
    "phone": "1234567890",
    "username": "johndoe"
}
```

**Validations:**
- ✅ Email format validation
- ✅ Password minimum 8 characters
- ✅ Username minimum 3 characters
- ✅ Full name required
- ✅ Phone required

---

### 1.2 User Login

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Email/Password login | ✅ Implemented | `auth_service.go:107` |
| Password verification | ✅ Implemented | `auth_service.go:113` |
| JWT token generation | ✅ Implemented | `auth_service.go:121` |
| Wallet info returned | ✅ Implemented | `auth_service.go:118` |
| Token expiry (7 days) | ✅ Implemented | `auth_service.go:140` |

---

### 1.3 Google OAuth Login

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Google ID login | ✅ Implemented | `auth_service.go:199` |
| Auto-registration for new users | ✅ Implemented | `auth_service.go:222` |
| Account linking (email match) | ✅ Implemented | `auth_service.go:210` |
| Avatar sync from Google | ✅ Implemented | `auth_service.go:215` |
| Auto wallet creation | ✅ Implemented | `auth_service.go:243` |

**Request Structure:**
```json
{
    "email": "user@gmail.com",
    "name": "John Doe",
    "google_id": "google_unique_id",
    "avatar": "https://..."
}
```

---

### 1.4 KYC Verification

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Date of birth submission | ✅ Implemented | `auth_service.go:185` |
| Address verification | ✅ Implemented | `auth_service.go:191` |
| SSN last 4 digits | ✅ Implemented | `auth_service.go:192` |
| Status update (pending → verified) | ✅ Implemented | `auth_service.go:193` |

**KYC Status Values:**
- `pending` - Initial state
- `verified` - KYC approved
- `rejected` - KYC rejected

---

### 1.5 User Profile

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Get current user | ✅ Implemented | `handlers/auth.go:52` |
| User response sanitization | ✅ Implemented | `models/user.go:74` |
| Wallet preloading | ✅ Implemented | `auth_service.go:167` |

**User Model Fields:**
```go
- ID (UUID)
- Email (unique, indexed)
- Phone (unique, indexed)
- PasswordHash (never exposed)
- FullName
- Username (unique, indexed)
- Avatar
- KYCStatus (pending/verified/rejected)
- DateOfBirth
- Address
- SSNLast4 (never exposed)
- PersonalQRData
- PersonalQRImage
- GoogleID
- AuthProvider (email/google)
- CreditScore (default: 650)
- CreatedAt, UpdatedAt, DeletedAt
```

---

## 2. Digital Wallet

### 2.1 Wallet Management

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Get wallet balance | ✅ Implemented | `wallet_service.go:32` |
| Currency support (USD) | ✅ Implemented | `models/wallet.go` |
| Active/Inactive status | ✅ Implemented | `wallet_service.go:46` |
| Decimal precision | ✅ Implemented | Uses `shopspring/decimal` |

---

### 2.2 Add Money (Deposit)

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Add from bank account | ✅ Implemented | `wallet_service.go:40` |
| Add from card | ✅ Implemented | `wallet_service.go:40` |
| Balance update | ✅ Implemented | `wallet_service.go:56` |
| Transaction record creation | ✅ Implemented | `wallet_service.go:63` |
| Atomic transaction (rollback) | ✅ Implemented | `wallet_service.go:53` |

**Request Structure:**
```json
{
    "amount": 100.50,
    "source": "bank",  // or "card"
    "description": "Optional note"
}
```

---

### 2.3 Withdraw Money

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Withdraw to bank | ✅ Implemented | `wallet_service.go:84` |
| Balance check | ✅ Implemented | `wallet_service.go:96` |
| Insufficient balance error | ✅ Implemented | `wallet_service.go:97` |
| Inactive wallet check | ✅ Implemented | `wallet_service.go:91` |
| Transaction record | ✅ Implemented | `wallet_service.go:111` |

**Request Structure:**
```json
{
    "amount": 50.00,
    "bank_account": "****1234",
    "description": "Optional note"
}
```

---

### 2.4 Transaction History

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Paginated transactions | ✅ Implemented | `wallet_service.go:132` |
| Limit/Offset support | ✅ Implemented | `handlers/wallet.go:85` |
| Max limit enforcement (100) | ✅ Implemented | `handlers/wallet.go:88` |
| Total count returned | ✅ Implemented | `wallet_service.go:139` |
| From/To user preloading | ✅ Implemented | `wallet_service.go:142` |
| Date range filtering | ✅ Implemented | `wallet_service.go:152` |

**Transaction Types:**
| Type | Description | Direction |
|------|-------------|-----------|
| `deposit` | Money added | Income |
| `withdraw` | Bank withdrawal | Expense |
| `p2p_send` | Sent to user | Expense |
| `p2p_receive` | Received from user | Income |
| `bill_pay` | Bill payment | Expense |
| `merchant` | QR/Merchant payment | Expense |
| `reward` | Cashback credit | Income |
| `refund` | Transaction refund | Income |

**Transaction Status:**
- `pending` - Processing
- `completed` - Successful
- `failed` - Failed
- `cancelled` - User cancelled

---

## 3. P2P Transfers

### 3.1 Send Money

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Send by username | ✅ Implemented | `transfer_service.go:44` |
| Send by email | ✅ Implemented | `transfer_service.go:44` |
| Send by phone | ✅ Implemented | `transfer_service.go:45` |
| Self-transfer prevention | ✅ Implemented | `transfer_service.go:51` |
| Insufficient balance check | ✅ Implemented | `transfer_service.go:63` |
| Atomic balance update | ✅ Implemented | `transfer_service.go:68` |
| Add note/description | ✅ Implemented | `transfer_service.go:93` |
| Auto cashback (1%) | ✅ Implemented | `transfer_service.go:107` |

**Request Structure:**
```json
{
    "recipient": "username_or_email_or_phone",
    "amount": 25.00,
    "note": "Thanks for lunch!"
}
```

---

### 3.2 Recent Contacts

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Get recent P2P contacts | ✅ Implemented | `transfer_service.go:115` |
| Deduplication | ✅ Implemented | `transfer_service.go:128` |
| Limit support | ✅ Implemented | `transfer_service.go:134` |
| Ordered by recency | ✅ Implemented | `transfer_service.go:119` |

---

### 3.3 User Search

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Search by username | ✅ Implemented | `transfer_service.go:147` |
| Search by email | ✅ Implemented | `transfer_service.go:147` |
| Search by full name | ✅ Implemented | `transfer_service.go:147` |
| Exclude current user | ✅ Implemented | `transfer_service.go:148` |
| Max 10 results | ✅ Implemented | `transfer_service.go:149` |
| LIKE pattern matching | ✅ Implemented | `transfer_service.go:145` |

---

## 4. Bill Payments

### 4.1 Bill Categories

| Category | Implemented |
|----------|-------------|
| Utilities (Electric, Water, Gas) | ✅ |
| Internet | ✅ |
| Phone/Mobile | ✅ |
| Subscriptions (Streaming) | ✅ |
| Insurance | ✅ |
| Rent | ✅ |

---

### 4.2 Billers

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Get all billers | ✅ Implemented | `bill_service.go:34` |
| Filter by category | ✅ Implemented | `bill_service.go:38` |
| Active status filtering | ✅ Implemented | `bill_service.go:36` |
| Account format validation | ✅ Implemented | `models/biller.go` |

**Pre-seeded Billers:**
| Biller Name | Category | Account Format |
|-------------|----------|----------------|
| Electric Company | utilities | 10 digits |
| Water Services | utilities | 8 digits |
| Gas Company | utilities | 10 digits |
| Internet Provider | internet | 12 digits |
| Mobile Carrier | phone | 10 digits |
| Streaming Service | subscription | alphanumeric |
| Insurance Co | insurance | 2 letters + 8 digits |
| Rent Payment | rent | 6 digits |

---

### 4.3 Pay Bill

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Biller validation | ✅ Implemented | `bill_service.go:56` |
| Balance check | ✅ Implemented | `bill_service.go:68` |
| Atomic payment | ✅ Implemented | `bill_service.go:73` |
| Transaction creation | ✅ Implemented | `bill_service.go:83` |
| Save biller option | ✅ Implemented | `bill_service.go:98` |
| Auto nickname | ✅ Implemented | `bill_service.go:105` |
| 2% cashback | ✅ Implemented | `bill_service.go:118` |

**Request Structure:**
```json
{
    "biller_id": "uuid",
    "account_number": "1234567890",
    "amount": 75.00,
    "save_biller": true,
    "nickname": "My Electric Bill"
}
```

---

### 4.4 Saved Billers

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| List saved billers | ✅ Implemented | `bill_service.go:44` |
| Remove saved biller | ✅ Implemented | `bill_service.go:123` |
| User scoping | ✅ Implemented | `bill_service.go:46` |
| Biller info preload | ✅ Implemented | `bill_service.go:48` |

---

## 5. Loans & EMI

### 5.1 Loan Offers

| Loan Type | Min Amount | Max Amount | Interest Rate | Term Range |
|-----------|------------|------------|---------------|------------|
| Personal Loan | $500 | $10,000 | 12.99% | 3-36 months |
| Emergency Loan | $100 | $2,000 | 15.99% | 1-12 months |
| Premium Loan | $5,000 | $50,000 | 8.99% | 12-60 months |

---

### 5.2 Credit Score Interest Rates

| Credit Score | Interest Rate |
|--------------|---------------|
| 750+ | 8.99% |
| 700-749 | 10.99% |
| 650-699 | 12.99% |
| 600-649 | 15.99% |
| Below 600 | 18.99% |

---

### 5.3 Loan Application

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Credit score check | ✅ Implemented | `loan_service.go:34` |
| EMI calculation | ✅ Implemented | `loan_service.go:40` |
| Total amount calculation | ✅ Implemented | `loan_service.go:41` |
| Loan record creation | ✅ Implemented | `loan_service.go:43` |
| Auto-approval (demo) | ✅ Implemented | `loan_service.go:63` |
| Funds disbursement | ✅ Implemented | `loan_service.go:88` |

**EMI Calculation Formula:**
```
EMI = P × r × (1+r)^n / ((1+r)^n - 1)

Where:
P = Principal amount
r = Monthly interest rate (annual rate / 12 / 100)
n = Number of months
```

---

### 5.4 Loan Management

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Get user loans | ✅ Implemented | `loan_service.go:108` |
| Get loan details | ✅ Implemented | `loan_service.go:115` |
| Loan status tracking | ✅ Implemented | `models/loan.go` |

**Loan Status:**
- `pending` - Application submitted
- `approved` - Approved, awaiting disbursement
- `active` - Disbursed, payments in progress
- `paid` - Fully repaid
- `rejected` - Application rejected
- `defaulted` - Payment default

---

### 5.5 EMI Payment

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Balance validation | ✅ Implemented | `loan_service.go:141` |
| Wallet deduction | ✅ Implemented | `loan_service.go:146` |
| Principal/Interest split | ✅ Implemented | `loan_service.go:160` |
| Payment record | ✅ Implemented | `loan_service.go:168` |
| Loan balance update | ✅ Implemented | `loan_service.go:184` |
| Next payment date | ✅ Implemented | `loan_service.go:187` |
| Auto status → paid | ✅ Implemented | `loan_service.go:190` |

---

## 6. Virtual Cards

### 6.1 Card Creation

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Visa-like card number | ✅ Implemented | `card_service.go:137` |
| CVV generation | ✅ Implemented | `card_service.go:149` |
| 3-year expiry | ✅ Implemented | `card_service.go:35` |
| Cardholder name | ✅ Implemented | `card_service.go:45` |
| Custom label | ✅ Implemented | `card_service.go:53` |
| Custom color | ✅ Implemented | `card_service.go:54` |
| Spending limits | ✅ Implemented | `card_service.go:48` |
| Daily limits | ✅ Implemented | `card_service.go:49` |

**Request Structure:**
```json
{
    "label": "Shopping Card",
    "color": "#4F46E5",
    "spending_limit": 5000,
    "daily_limit": 500
}
```

---

### 6.2 Card Security

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Card number masking | ✅ Implemented | `card_service.go:159` |
| Full details with OTP | ✅ Implemented | `card_service.go:89` |
| CVV hidden by default | ✅ Implemented | Always hidden |
| Card freezing | ✅ Implemented | `card_service.go:101` |

**Card Number Display:**
- Masked: `**** **** **** 1234`
- Full: `4123 4567 8901 2345` (OTP required)

---

### 6.3 Card Management

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| List all cards | ✅ Implemented | `card_service.go:65` |
| Get card details | ✅ Implemented | `card_service.go:78` |
| Freeze/Unfreeze | ✅ Implemented | `card_service.go:101` |
| Update limits | ✅ Implemented | `card_service.go:113` |
| Delete/Close card | ✅ Implemented | `card_service.go:129` |

**Card Status:**
- `active` - Card is usable
- `frozen` - Temporarily disabled
- `closed` - Permanently closed

---

## 7. Merchant & QR Payments

### 7.1 Merchant Registration

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Register as merchant | ✅ Implemented | `qr_service.go:246` |
| Business name | ✅ Implemented | `qr_service.go:248` |
| Business category | ✅ Implemented | `qr_service.go:249` |
| Default fee (1.5%) | ✅ Implemented | `qr_service.go:250` |

---

### 7.2 QR Code Types

| Type | Description | Expiry | Usage |
|------|-------------|--------|-------|
| `static` | Reusable, payer enters amount | Never | Shop counter |
| `dynamic` | One-time, fixed amount | 15 minutes | Invoices |

---

### 7.3 QR Code Generation

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Static QR codes | ✅ Implemented | `qr_service.go:43` |
| Dynamic QR codes | ✅ Implemented | `qr_service.go:59` |
| Amount embedding | ✅ Implemented | `qr_service.go:53` |
| Custom labels | ✅ Implemented | `qr_service.go:54` |
| QR image (Base64 PNG) | ✅ Implemented | `qr_service.go:80` |
| Expiry setting | ✅ Implemented | `qr_service.go:61` |

**QR Data Format:**
```
flowpay://pay/{merchant_id}/{unique_code}
```

---

### 7.4 QR Payment

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Scan QR data | ✅ Implemented | `qr_service.go:102` |
| QR validation | ✅ Implemented | `qr_service.go:105` |
| Expiry check | ✅ Implemented | `qr_service.go:110` |
| Balance check | ✅ Implemented | `qr_service.go:130` |
| Fee calculation | ✅ Implemented | `qr_service.go:141` |
| Atomic payment | ✅ Implemented | `qr_service.go:145` |
| Merchant credit | ✅ Implemented | `qr_service.go:155` |
| Scan count update | ✅ Implemented | `qr_service.go:181` |
| One-time use (dynamic) | ✅ Implemented | `qr_service.go:183` |
| 1.5% cashback | ✅ Implemented | `qr_service.go:190` |

---

### 7.5 Merchant Dashboard

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Total revenue | ✅ Implemented | `qr_service.go:207` |
| Today's revenue | ✅ Implemented | `qr_service.go:215` |
| Total transactions | ✅ Implemented | `qr_service.go:224` |
| Today's transactions | ✅ Implemented | `qr_service.go:230` |
| Active QR codes count | ✅ Implemented | `qr_service.go:237` |

---

## 8. Rewards & Cashback

### 8.1 Cashback Rates

| Transaction Type | Cashback Rate | Points Rate |
|------------------|---------------|-------------|
| P2P Transfer | 1% | 10 points/$ |
| Bill Payment | 2% | 10 points/$ |
| Merchant Payment | 1.5% | 10 points/$ |

---

### 8.2 Reward Features

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Auto cashback award | ✅ Implemented | `reward_service.go:19` |
| Points calculation | ✅ Implemented | `reward_service.go:21` |
| Wallet credit | ✅ Implemented | `reward_service.go:43` |
| Cashback transaction | ✅ Implemented | `reward_service.go:49` |
| Async processing | ✅ Implemented | Using goroutines |

---

### 8.3 Rewards Summary

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Total points | ✅ Implemented | `reward_service.go:65` |
| Total cashback | ✅ Implemented | `reward_service.go:72` |
| Lifetime cashback | ✅ Implemented | `reward_service.go:80` |
| Today's transactions | ✅ Implemented | `reward_service.go:88` |
| Average cashback rate | ✅ Implemented | `reward_service.go:96` |

---

### 8.4 Rewards History

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Paginated history | ✅ Implemented | `reward_service.go:101` |
| Transaction linking | ✅ Implemented | `reward_service.go:109` |
| Ordered by date | ✅ Implemented | `reward_service.go:110` |

---

### 8.5 Promotional Offers

| Offer | Cashback | Category | Duration |
|-------|----------|----------|----------|
| 5% Cashback on Bills | 5% | bills | 7 days |
| 3% on P2P Transfers | 3% | p2p | 3 days |
| Double Points Weekend | 2x | all | 2 days |
| First Merchant Payment | $5 | merchant | 30 days |

---

## 9. Stock Trading Platform

### 9.1 Trading Account KYC

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Date of birth (18+ check) | ✅ Implemented | `trading_service.go:71` |
| SSN (9 digits) | ✅ Implemented | `trading_service.go:85` |
| SSN hashing (SHA-256) | ✅ Implemented | `trading_service.go:95` |
| Security questions (2) | ✅ Implemented | `trading_service.go:102` |
| Answer hashing | ✅ Implemented | `trading_service.go:416` |
| Risk acknowledgment | ✅ Implemented | `trading_service.go:90` |

---

### 9.2 Stock Data (RapidAPI Integration)

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Stock search | ✅ Implemented | `stock_service.go:318` |
| Real-time quotes | ✅ Implemented | `stock_service.go:107` |
| Stock details | ✅ Implemented | `stock_service.go:142` |
| Historical charts | ✅ Implemented | `stock_service.go:182` |
| Market summary | ✅ Implemented | `stock_service.go:490` |
| India most active | ✅ Implemented | `stock_service.go:496` |
| Corporate actions | ✅ Implemented | `stock_service.go:231` |
| All stocks list | ✅ Implemented | `stock_service.go:361` |

---

### 9.3 Stock Quote Data

| Field | Available |
|-------|-----------|
| Symbol | ✅ |
| Company Name | ✅ |
| Current Price | ✅ |
| Open Price | ✅ |
| High | ✅ |
| Low | ✅ |
| Previous Close | ✅ |
| Volume | ✅ |
| Change (absolute) | ✅ |
| Change (percent) | ✅ |
| Latest Trading Day | ✅ |
| Timestamp | ✅ |

---

### 9.4 Stock Details

| Field | Available |
|-------|-----------|
| Symbol | ✅ |
| Company Name | ✅ |
| Description | ✅ |
| Exchange | ✅ |
| Sector | ✅ |
| Industry | ✅ |
| Market Cap | ✅ |
| P/E Ratio | ✅ |
| EPS | ✅ |
| Beta | ✅ |
| 52-Week High | ✅ |
| 52-Week Low | ✅ |
| 50-Day Moving Avg | ✅ |
| 200-Day Moving Avg | ✅ |

---

### 9.5 Trade Execution

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Buy orders | ✅ Implemented | `trading_service.go:186` |
| Sell orders | ✅ Implemented | `trading_service.go:196` |
| Market price fetch | ✅ Implemented | `trading_service.go:164` |
| Buying power check | ✅ Implemented | `trading_service.go:189` |
| Position check (sell) | ✅ Implemented | `trading_service.go:200` |
| Instant fill | ✅ Implemented | `trading_service.go:221` |
| Zero commission | ✅ Implemented | `trading_service.go:172` |
| Position update | ✅ Implemented | `trading_service.go:227` |
| Portfolio recalc | ✅ Implemented | `trading_service.go:237` |

**Request Structure:**
```json
{
    "symbol": "RELIANCE",
    "quantity": 10,
    "type": "buy"  // or "sell"
}
```

---

### 9.6 Portfolio Management

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| List positions | ✅ Implemented | `trading_service.go:117` |
| Real-time prices | ✅ Implemented | `trading_service.go:131` |
| Market value calc | ✅ Implemented | `trading_service.go:134` |
| Unrealized P/L | ✅ Implemented | `trading_service.go:136` |
| P/L percentage | ✅ Implemented | `trading_service.go:138` |
| Weighted avg cost | ✅ Implemented | `trading_service.go:260` |

---

### 9.7 Watchlist

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Add to watchlist | ✅ Implemented | `trading_service.go:309` |
| Remove from watchlist | ✅ Implemented | `trading_service.go:375` |
| Get watchlist | ✅ Implemented | `trading_service.go:343` |
| Real-time prices | ✅ Implemented | `trading_service.go:357` |
| Duplicate prevention | ✅ Implemented | `trading_service.go:317` |

---

### 9.8 Trade History

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| All trades | ✅ Implemented | `trading_service.go:385` |
| Ordered by date | ✅ Implemented | `trading_service.go:393` |
| Limit support | ✅ Implemented | `trading_service.go:394` |

---

### 9.9 Funds Management

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Deposit to trading | ✅ Implemented | `trading_service.go:401` |
| Buying power increase | ✅ Implemented | `trading_service.go:407` |

---

### 9.10 Caching Strategy

| Data Type | Cache TTL |
|-----------|-----------|
| Stock Quotes | 1 minute |
| Chart Data | 5 minutes |
| Stock Details | 1 hour |
| News | 15 minutes |
| All Stocks List | 5 minutes |
| India Quotes | 1 minute |

---

## 10. Statement Generation

### 10.1 CSV Statement

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Account holder info | ✅ Implemented | `statement_service.go:87` |
| Date range | ✅ Implemented | `statement_service.go:88` |
| Current balance | ✅ Implemented | `statement_service.go:89` |
| Income/Expense totals | ✅ Implemented | `statement_service.go:93` |
| Net change | ✅ Implemented | `statement_service.go:96` |
| Transaction list | ✅ Implemented | `statement_service.go:102` |

---

### 10.2 PDF/Text Statement

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Formatted header | ✅ Implemented | `statement_service.go:129` |
| Opening balance | ✅ Implemented | `statement_service.go:142` |
| Credits/Debits | ✅ Implemented | `statement_service.go:143` |
| Closing balance | ✅ Implemented | `statement_service.go:145` |
| Detailed transactions | ✅ Implemented | `statement_service.go:151` |

---

## 11. OTP System

### 11.1 OTP Generation

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| 6-digit random code | ✅ Implemented | `otp_service.go:27` |
| Cryptographic randomness | ✅ Implemented | `otp_service.go:96` |
| 5-minute expiry | ✅ Implemented | `otp_service.go:38` |
| Purpose tracking | ✅ Implemented | `otp_service.go:36` |
| Reference ID | ✅ Implemented | `otp_service.go:37` |
| Console logging (demo) | ✅ Implemented | `otp_service.go:48` |

**OTP Purposes:**
- `transfer` - P2P transfer verification
- `card` - Card details viewing
- `login` - 2FA login (future)

---

### 11.2 OTP Verification

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Code validation | ✅ Implemented | `otp_service.go:74` |
| Expiry check | ✅ Implemented | `otp_service.go:66` |
| Attempt limiting (3) | ✅ Implemented | `otp_service.go:70` |
| Mark as verified | ✅ Implemented | `otp_service.go:79` |

---

## 12. Frontend Features

### 12.1 Dashboard Component

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Time-based greeting | ✅ Implemented | `dashboard.component.ts:508` |
| Balance display | ✅ Implemented | Template line 37 |
| KYC badge | ✅ Implemented | Template line 22 |
| Stats cards (income/spending/points) | ✅ Implemented | Template line 55 |
| Quick actions grid | ✅ Implemented | Template line 85 |
| Recent transactions | ✅ Implemented | Template line 140 |
| Loading states | ✅ Implemented | Template line 128 |
| Empty states | ✅ Implemented | Template line 133 |
| Promo banner | ✅ Implemented | Template line 159 |

---

### 12.2 Trading Component

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Account status check | ✅ Implemented | `trading.component.ts` |
| KYC verification modal | ✅ Implemented | `` |
| Stock search | ✅ Implemented | `onSearchInput()` |
| Real-time chart (TradingView) | ✅ Implemented | `lightweight-charts` |
| Market indices banner | ✅ Implemented | `` |
| Portfolio positions | ✅ Implemented | `loadPositions()` |
| Trade execution modal | ✅ Implemented | `` |
| Watchlist management | ✅ Implemented | `loadWatchlist()` |
| Price alerts | ✅ Implemented | `PriceAlert` interface |
| Keyboard shortcuts | ✅ Implemented | `handleShortcuts()` |
| Panel toggling | ✅ Implemented | `toggleLeftPanel()` |
| Panel state persistence | ✅ Implemented | `persistPanelState()` |
| Fund deposits | ✅ Implemented | `depositFunds()` |

---

### 12.3 Loans Component

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Loan offers list | ✅ Implemented | `loadOffers()` |
| Active loans list | ✅ Implemented | `loadLoans()` |
| Credit score display | ✅ Implemented | `getCreditStatus()` |
| EMI calculator | ✅ Implemented | `calculateEMI()` |
| Loan application | ✅ Implemented | `submitApplication()` |
| EMI payment | ✅ Implemented | `payEMI()` |

---

### 12.4 Cards Component

| Feature | Implementation | File Location |
|---------|----------------|---------------|
| Card list | ✅ Implemented | `loadCards()` |
| Card creation modal | ✅ Implemented | `createCard()` |
| View full details (OTP) | ✅ Implemented | `viewFullDetails()` |
| OTP verification modal | ✅ Implemented | `verifyOTP()` |
| Freeze/Unfreeze toggle | ✅ Implemented | `toggleFreeze()` |
| Delete card | ✅ Implemented | `deleteCard()` |

---

### 12.5 Frontend Services

| Service | Features |
|---------|----------|
| `AuthService` | Login, register, Google auth, getMe, logout, token management |
| `WalletService` | Balance, add money, withdraw, transactions, statement download |
| `TransferService` | Send money, contacts, search |
| `BillService` | Categories, billers, pay, saved billers |
| `LoanService` | Offers, apply, loans, payments |
| `CardService` | CRUD, freeze, OTP, details |
| `MerchantService` | Register, QR, dashboard |
| `RewardService` | Summary, history |
| `StockService` | Search, quote, details, chart, news, indices |
| `TradingService` | Account, verify, deposit, portfolio, trade, watchlist |

---

## 13. Security Features

### 13.1 Authentication Security

| Feature | Implementation |
|---------|----------------|
| Password hashing (bcrypt) | ✅ |
| JWT token signing (HS256) | ✅ |
| Token expiry (7 days) | ✅ |
| Authorization header validation | ✅ |
| Bearer token format check | ✅ |

---

### 13.2 Data Protection

| Feature | Implementation |
|---------|----------------|
| SSN hashing (SHA-256) | ✅ |
| Card number masking | ✅ |
| OTP with limited attempts | ✅ |
| Password never exposed | ✅ |
| Sensitive fields hidden in JSON | ✅ |

---

### 13.3 API Security

| Feature | Implementation |
|---------|----------------|
| CORS configuration | ✅ |
| Protected routes middleware | ✅ |
| Input validation (Gin binding) | ✅ |
| SQL injection prevention (GORM) | ✅ |
| Atomic transactions (rollback) | ✅ |

---

## 14. External Integrations

### 14.1 RapidAPI (Stock Data)

| Integration | Endpoint | Purpose |
|-------------|----------|---------|
| Stock Search | `/stock` | Find stocks by name |
| Stock Quote | `/stock` | Real-time price |
| Historical Data | `/historical_data` | Chart data |
| Most Active | `/NSE_most_active` | Market movers |
| Corporate Actions | `/corporate_actions` | Dividends, splits |

**Configuration:**
```env
RAPIDAPI_KEY=your_key
RAPIDAPI_HOST=indian-stock-exchange-api2.p.rapidapi.com
```

---

### 14.2 Google OAuth

| Integration | Purpose |
|-------------|---------|
| Google ID verification | User authentication |
| Profile sync | Name, email, avatar |
| Account linking | Connect existing accounts |

---

### 14.3 QR Code Generation

| Library | Purpose |
|---------|---------|
| `skip2/go-qrcode` | Generate QR images |
| Base64 encoding | Embed in JSON response |

---

## 📊 Implementation Summary

### Backend Services
- **10 Services** in main backend
- **2 Services** in trading backend
- **8 Handlers** for API routing

### Database Models
- **15 Models** across both databases
- Auto-migration on startup
- Seeded data for billers and loan offers

### Frontend Components
- **11 Feature components**
- **11 Angular services**
- Standalone components (Angular 19)
- Reactive signals for state

### API Endpoints
- **40+ RESTful endpoints**
- Protected & public routes
- Full CRUD operations

---

*Feature Guide completed on January 29, 2026*
*FlowPay v1.0.0*
