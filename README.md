# GLMS – Global Logistics Management System
### TechMove Logistics | Part 2 – ASP.NET Core MVC Monolith

---

## Technology Stack
- **Framework**: ASP.NET Core 8 MVC
- **ORM**: Entity Framework Core 8 (SQL Server)
- **Testing**: xUnit + Moq + EF Core InMemory
- **UI**: PowerBI-inspired custom CSS + Font Awesome 6

---

## Prerequisites
- Visual Studio 2022 (v17.8+)
- .NET 8 SDK
- SQL Server or SQL Server Express (LocalDB works)
- Internet access (for live currency rate API)

---

## Setup Instructions

### 1. Clone / Open Solution
```
Open GLMS.sln in Visual Studio 2022
```

### 2. Restore NuGet Packages
Visual Studio will restore automatically. Or run:
```bash
dotnet restore
```

### 3. Configure Database Connection
Edit `GLMS/appsettings.json`:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=GLMS_DB;Trusted_Connection=True;"
  }
}
```
For full SQL Server, replace with your connection string.

### 4. Apply Migrations & Seed Data
The app applies migrations automatically on startup via `db.Database.Migrate()` in Program.cs.

OR run manually in Package Manager Console:
```
Update-Database
```

OR via CLI:
```bash
cd GLMS
dotnet ef database update
```

### 5. Run the Application
Press **F5** in Visual Studio or:
```bash
cd GLMS
dotnet run
```
The app opens at `https://localhost:5001` (or the port shown in console).

---

## Running Unit Tests

### Via Visual Studio Test Explorer:
1. Build the solution (Ctrl+Shift+B)
2. Open Test > Test Explorer
3. Click "Run All Tests"

### Via CLI:
```bash
cd GLMS.Tests
dotnet test --verbosity normal
```

---

## Key Features

### ✅ Contract Management Hub
- Create/Edit/Delete contracts linked to clients
- Automated status tracking: Draft → Active → Expired → On Hold
- LINQ-powered search/filter by date range and status
- PDF signed agreement upload, storage & download

### ✅ Service Request Processing
- Workflow enforcement: requests BLOCKED on Expired, On Hold, and Draft contracts
- Only Active contracts accept new service requests

### ✅ Financial Integration (USD → ZAR)
- Live exchange rate from `open.er-api.com` (no API key needed)
- Auto-calculates ZAR as user types USD amount
- Rate cached for 1 hour, fallback rate R18.75 if API unavailable
- Exchange rate stored with each service request for audit trail

### ✅ File Handling
- PDF upload with drag-and-drop UI
- Magic byte validation (not just extension check)
- Max 10 MB limit
- Downloadable via browser

### ✅ Unit Tests (xUnit)
| Test Class | Tests |
|---|---|
| CurrencyServiceTests | 10 tests – USD/ZAR math, edge cases, Theory tests |
| FileServiceTests | 8 tests – PDF validation, blocked extensions, size limits |
| WorkflowTests | 9 tests – contract status rules, LINQ filters, cascade deletes |

---

## Project Structure
```
GLMS/
├── GLMS/                        # Main web application
│   ├── Controllers/
│   │   ├── HomeController.cs    # Dashboard
│   │   ├── ClientsController.cs # Full CRUD
│   │   ├── ContractsController.cs # Full CRUD + PDF upload
│   │   └── ServiceRequestsController.cs # Full CRUD + currency
│   ├── Models/
│   │   ├── Client.cs
│   │   ├── Contract.cs          # ContractStatus + ServiceLevel enums
│   │   └── ServiceRequest.cs    # RequestStatus enum
│   ├── Data/
│   │   └── AppDbContext.cs      # EF Core DbContext + seed data
│   ├── Services/
│   │   ├── CurrencyService.cs   # Live USD→ZAR via API
│   │   └── FileService.cs       # PDF upload + validation
│   ├── ViewModels/
│   │   └── ViewModels.cs        # Dashboard, Filter, CreateVM
│   ├── Views/
│   │   ├── Home/Index.cshtml    # PowerBI Dashboard
│   │   ├── Clients/             # Index, Create, Edit, Details, Delete
│   │   ├── Contracts/           # Index, Create, Edit, Details, Delete
│   │   ├── ServiceRequests/     # Index, Create, Edit, Details, Delete
│   │   └── Shared/_Layout.cshtml
│   ├── wwwroot/
│   │   ├── css/site.css         # Full PowerBI design system
│   │   └── js/site.js           # Currency converter, file upload
│   ├── Migrations/              # EF Core migration + snapshot
│   ├── Program.cs               # DI + middleware config
│   └── appsettings.json
│
└── GLMS.Tests/                  # xUnit test project
    ├── GLMSTests.cs             # All unit tests
    └── GLMS.Tests.csproj
```

---

## Database Schema
```
Clients (Id, Name, Email, Phone, Region, Address, CreatedAt)
    │
    └──< Contracts (Id, ClientId→Clients, Title, StartDate, EndDate,
                    Status, ServiceLevel, Notes,
                    SignedAgreementPath, SignedAgreementFileName, CreatedAt)
              │
              └──< ServiceRequests (Id, ContractId→Contracts, Description,
                                    CostUSD, CostZAR, ExchangeRateUsed,
                                    Status, Origin, Destination, Notes, CreatedAt)
```

---

## Architecture Notes
This is the **Part 2 Monolith** — a single deployable unit:
- UI (Views), Business Logic (Controllers/Services), and Data (EF Core) are all in one project
- This establishes proven core functionality before Part 3's containerisation with Docker
- Aligns with TOGAF's phased Architecture Development Method (ADM)
