# 🔐 Secure Refresh Token Handling

### Node.js Backend + Flutter Client Integration

This repository demonstrates a **complete, best-practice Refresh Token authentication flow** using:

* **Node.js (Express)** for backend APIs
* **Flutter** for client-side integration
* **JWT-based authentication** with secure token rotation
* **Automatic access token refresh on expiry**

This repo is designed to show **how authentication should actually work in real apps** 🚀

---

## ✨ What This Repo Covers

### Backend (Node.js)

* ✅ Access Token + Refresh Token flow
* ✅ JWT authentication
* ✅ Refresh token rotation
* ✅ Token invalidation on logout
* ✅ Secure API design

### Frontend (Flutter)

* ✅ API integration with Node.js backend
* ✅ Automatic access token refresh
* ✅ Centralized API client
* ✅ Request retry after token refresh
* ✅ Graceful logout on refresh token expiry

---

## 🧠 Why This Architecture?

Most apps fail authentication because they:

* Use long-lived access tokens ❌
* Don’t handle token expiry properly ❌
* Break UX when token expires ❌

This repo solves that by:

* Using **short-lived access tokens**
* Using **refresh tokens for silent re-auth**
* Handling expiry **without user interruption**

---

## 🔄 Authentication Flow (High Level)

```
Login
  ↓
Access Token (short-lived)
Refresh Token (long-lived)
  ↓
API request fails with 401
  ↓
Refresh Token API
  ↓
New Access Token
  ↓
Retry original API
```

---

## 📂 Project Structure

```
.
├── server
│     ├── controllers
│     ├── routes
│     ├── middleware
│     ├── utils
│     ├── app.js
│     └── package.json
│
├── flutter_app
│   ├──lib/
│   │   ├── core/
│   │   ├── services/
│   │   ├── api_endpoint.dart
│   │   ├── api_service.dart
│   │   ├── app_navigator.dart
│   │   ├── auth_api_service.dart
│   │   ├── pref_service.dart
│   │   └── session_manager.dart
│   │
│   ├── utils/
│   │   └── amount_formatter.dart
│   │
│   └── model/
│       ├── dashboard_overview.dart
│       ├── login_response_model.dart
│       └── recent_transaction.dart
│
├── view/
│   ├── dashboard_screen.dart
│   ├── loading_screen.dart
│   ├── login_screen.dart
│   └── profile_screen.dart
│
└── main.dart

```

---

## 🔑 Backend APIs (Node.js)

### 1️⃣ Login

**POST** `/api/login`

Returns:

```json
{
  "accessToken": "jwt-access-token",
  "refreshToken": "jwt-refresh-token"
}
```

---

### 2️⃣ Refresh Token

**POST** `/api/refresh`

* Validates refresh token
* Rotates refresh token
* Issues new access token

```json
{
  "accessToken": "new-access-token",
  "refreshToken": "new-refresh-token"
}
```

---

### 3️⃣ Protected API

**GET** `/api/dashboard_overview`

* Requires valid access token
* Returns 401 if expired

---

### 4️⃣ Logout

**POST** `/api/logout`

* Invalidates refresh token
* Forces re-login

---

## 📱 Flutter Integration (Client Side)

### 🔐 Token Storage

* Access token stored in memory / secure storage
* Refresh token stored securely
* Centralized token access

---

### 🌐 API Client with Interceptor

Flutter uses a **network interceptor** to:

* Attach access token to every request
* Detect `401 Unauthorized`
* Automatically call refresh API
* Retry failed request

**Flow inside Flutter:**

1. API call made
2. Access token expired → `401`
3. Refresh token API called
4. Tokens updated
5. Original API retried automatically

---

### 🧩 Flutter Token Refresh Logic (Concept)

```dart

    http.Response response = await http.get(uri, headers: headers);
    _log(uri.toString(), jsonEncode(headers), "GET", "", response);

    // If access token expired
    if (response.statusCode == 401) {
      final refreshed = await _refreshToken();

      // Navigate to login if refresh failed
      if (!refreshed) {
        await SessionManager.instance.logout();
        AppNavigator.navKey.currentState?.pushNamedAndRemoveUntil(
          "/login",
          (route) => false,
        );
        throw Exception("Session expired. Please login again.");
      }

      // Retry after refresh
      final newToken = await PrefService.getAccessToken();
      final newHeaders = _headers(token: newToken);

      response = await http.get(uri, headers: newHeaders);
      _log(
        uri.toString(),
        jsonEncode(newHeaders),
        "GET (Retry After Refresh)",
        "",
        response,
      );
    }
```

---

## 🚫 When User Is Logged Out Automatically

Flutter will force logout when:

* Refresh token expires
* Refresh token is invalid
* Refresh API returns 401 / 403

This ensures **maximum security**.

---

## 🔐 Security Best Practices Used

* Short-lived access tokens
* Long-lived refresh tokens
* Refresh token rotation
* Secure token storage
* No token leakage to UI
* Logout invalidates refresh token

---

## ⚙️ Environment Variables (Backend)

```env
PORT=3000
ACCESS_TOKEN_SECRET=your_access_secret
REFRESH_TOKEN_SECRET=your_refresh_secret
ACCESS_TOKEN_EXPIRE=15m
REFRESH_TOKEN_EXPIRE=7d
```

---

## 🚀 Getting Started

### Backend

```bash
cd backend
npm install
npm run dev
```

### Flutter

```bash
cd flutter_app
flutter pub get
flutter run
```

---

## 🧪 Recommended Testing Flow

1. Login from Flutter
2. Call protected API
3. Wait for access token expiry
4. API auto-refreshes token
5. API retries automatically
6. Logout
7. Refresh token becomes invalid

---

## 📌 Ideal For

* Flutter developers
* Node.js backend developers
* Real-world auth implementations
* Interview preparation
* Production-ready authentication systems

---

## 🤝 Contributing

PRs and suggestions are welcome 🙌
Feel free to improve security, structure, or add tests.

---

## ⭐ Support

If this repo helped you, **give it a star ⭐**
and share it with other developers!

---

Happy coding 🚀
