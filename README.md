This `README.md` provides a comprehensive overview of your **Online Auction System**, a distributed microservices architecture built with **ASP.NET Core 8**. It covers the purpose of each service, the database schema, security implementation, and how the services interact.

---

# Online Auction System - Microservices Backend

This repository contains the backend microservices for a full-stack Online Auction System. The system is designed using a decoupled architecture where individual services handle Identity, Auctions, Bidding, and Payments.

## 🏗 System Architecture

The system consists of four primary microservices communicating via RESTful APIs and shared JWT security protocols.

| Service | Responsibility | Port (Local) |
| --- | --- | --- |
| **Identity Service** | User registration, login, and JWT generation (Roles: Admin, Seller, Buyer). | `5000` |
| **Auction Service** | Managing auction listings, product details, and image uploads. | `5115` |
| **Bidding Service** | Processing real-time bids and tracking bid history. | `5174` |
| **Payment Service** | Simulating transactions for won auctions. | `5201` |

---

## 🔐 Security & Authentication

The system uses **JWT (JSON Web Token)** for stateless authentication across all services.

* **Issuer:** `OnlineAuctionIdentityService`
* **Audience:** `OnlineAuctionSystemClients`
* **Claims:** Tokens include `sub` (User ID), `unique_name` (Username), and `role` (Buyer/Seller/Admin).
* **Policies:**
* `SellerOnly`: Required to create auctions.
* `BuyerOnly`: Required to place bids and process payments.



---

## 📂 Service Breakdown

### 1. Identity Service

The core authority for user management.

* **Tech Stack:** ASP.NET Core Identity, Entity Framework Core, SQL Server.
* **Key Endpoints:**
* `POST /api/Account/register`: Creates a new user with a specific role.
* `POST /api/Account/login`: Validates credentials and returns a JWT.
* `GET /api/Account/users`: Admin-only view of all registered users.


* **Database:** `OnlineAuctionIdentityDB`

### 2. Auction Service

Handles the lifecycle of an auction.

* **Features:**
* Image Upload support (stored in `wwwroot/uploads`).
* Automatic StartTime and calculated EndTime based on duration.


* **Key Endpoints:**
* `GET /api/Auctions`: Publicly view all live auctions.
* `POST /api/Auctions`: Create a listing (Seller only).
* `PUT /api/Auctions/highestBid/{id}`: Internal endpoint used by Bidding Service to update the leader.


* **Database:** `OnlineAuctionServiceDB`

### 3. Bidding Service

Manages the competitive bidding process.

* **Logic:**
* Before saving a bid, it calls the **Auction Service** to verify if the auction is still "Live."
* Once a bid is saved, it triggers an update to the Auction Service to refresh the `HighestBid`.


* **Key Endpoints:**
* `POST /api/Bids`: Place a bid (Buyer only).
* `GET /api/Bids/{auctionId}`: View bid history for a specific item.


* **Database:** `OnlineAuctionBiddingDB`

### 4. Payment Service

Simulates the financial conclusion of an auction.

* **Logic:**
* Validates that the requester is the `HighestBidder`.
* Simulates payment gateway logic (fails if card number ends in `0`).


* **Key Endpoints:**
* `POST /api/Payments/process`: Process payment for a won auction.
* `GET /api/Payments/auction/{id}`: Retrieve receipt details.


* **Database:** `OnlineAuctionPaymentDB`

---

## 🛠 Database Configuration

The system uses **SQL Server (LocalDB/SQLEXPRESS)**. Each service has its own connection string in `appsettings.json`.

### Main Tables

* **AspNetUsers:** (Identity) Stores credentials and roles.
* **Auctions:** (Auction) Contains `ProductName`, `StartPrice`, `HighestBid`, and `Status`.
* **Bids:** (Bidding) Audit log of all `Amount` and `Timestamp` entries per `AuctionId`.
* **Transactions:** (Payment) Stores successful payment records.

---

## 🚀 Getting Started

### Prerequisites

* .NET 8 SDK
* SQL Server Express
* Visual Studio 2022 / VS Code

### Setup Steps

1. **Clone the repository.**
2. **Update Connection Strings:** In each service's `appsettings.json`, ensure the `Server` name matches your local SQL instance (e.g., `Server=localhost\SQLEXPRESS`).
3. **Apply Migrations:** Run the following command in the terminal for **each** service folder:
```bash
dotnet ef database update

```


4. **Run Services:** Start all four projects simultaneously using the Visual Studio "Multiple Startup Projects" feature or by running `dotnet run` in four separate terminals.

### CORS Configuration

The backend is pre-configured to allow requests from the React frontend running on:
`http://localhost:5173`

---

## 📝 Developer Notes

* **Internal Communication:** Services use `HttpClient` to talk to each other. Ensure the ports in `appsettings.json` under `AuctionService:Url` match the actual running port of the Auction Service.
* **Debugging:** `Console.WriteLine` debug logs are implemented in the `AuctionsController` to track incoming bid updates from the Bidding service.
