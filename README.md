# Refrigerator Repair Esfahan - Final Functional Build

ASP.NET Core 8 / Razor Pages / Clean Architecture / EF Core / MySQL.

## Architecture

Web -> Application -> Domain
Infrastructure -> Application + Domain

The UI is connected to real Application services and database entities. The customer flow is:

ServiceCatalog -> RepairRequest -> TechnicianOffer -> RepairJob -> RepairParts / WalletTransactions -> History

Admin and Technician dashboards read live database data.

## Production

1. Configure `ConnectionStrings__DefaultConnection`.
2. Configure `Payment__MerchantId` as a secret.
3. Configure `Payment__PublicBaseUrl` as the real HTTPS domain.
4. Run the app. EF Core applies the MySQL migration automatically.
5. In Development, FakePaymentGateway is used. In Production, ZarinPalGateway is used.

See `docs/PRODUCTION_SETUP.md` and `docs/ARCHITECTURE.md`.
