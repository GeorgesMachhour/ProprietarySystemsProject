# Active Directory Lightweight Directory Services (AD LDS) Project

## 📘 Project Overview
This project implements a robust enterprise-level directory service using Active Directory Lightweight Directory Services (AD LDS) with a modern ASP.NET Core web application for management and external access.
(We chose ASP.NET over WinForms and other Windows Desktop App GUI because it is easier to access taking in consideration that it is web-based and cross-platform.)
## 🎯 Project Objectives
- Create a scalable lightweight directory service
- Develop a secure web-based user management interface
- Implement LDAP-based authentication and user operations
- Provide flexible external user access mechanism

## 🖥️ System Requirements

### Hardware Requirements
- **Server**:
  - Processor: Intel Xeon or AMD EPYC (4 cores minimum)
  - RAM: 16GB (minimum), 32GB (recommended)
  - Storage: 
    - 100GB SSD for OS and AD LDS
    - Additional storage for logs and backups
  - Network: Gigabit Ethernet adapter

### Software Prerequisites
- **Operating System**:
  - Windows Server 2022 (Standard or Datacenter Edition)
  - Fully updated with latest Windows Updates

- **Development Environment**:
  - Visual Studio 2022 Enterprise or Professional
  - .NET 7.0 or .NET 8.0 SDK
  - SQL Server 2022 (optional, for extended data storage)

## 🚀 Comprehensive Installation Guide

### 1. Windows Server 2022 Preparation

#### 1.1 Initial Server Configuration
1. Install Windows Server 2022
2. Configure network settings:
   ```powershell
   # Set Static IP Example
   New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "192.168.1.100" -PrefixLength 24 -DefaultGateway "192.168.1.1"
   Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses ("8.8.8.8", "1.1.1.1")
   ```

3. Rename computer (optional but recommended)
   ```powershell
   Rename-Computer -NewName "ADLDS-SERVER"
   Restart-Computer
   ```

4. Disable IE Enhanced Security Configuration
   - Open Server Manager
   - Configure IE Enhanced Security to "Off" for Administrators

#### 1.2 Install AD LDS Role
```powershell
# Install AD LDS and Management Tools
Install-WindowsFeature -Name "ADLDS" -IncludeManagementTools -Restart

# Verify Installation
Get-WindowsFeature | Where-Object {$_.Name -like "*ADLDS*"}
```

#### 1.3 Configure Firewall
```powershell
# Create Firewall Rules for LDAP
New-NetFirewallRule -Name "LDAP-In" -DisplayName "LDAP Inbound" `
    -Protocol TCP -LocalPort 389 -Action Allow -Enabled True
New-NetFirewallRule -Name "LDAPS-In" -DisplayName "LDAPS Inbound" `
    -Protocol TCP -LocalPort 636 -Action Allow -Enabled True
```

### 2. AD LDS Instance Configuration

#### 2.1 Create LDS Instance via PowerShell
```powershell
# Example Instance Creation
$instanceName = "MyCompanyLDS"
$port = 389
$bindingIP = "0.0.0.0"

# Add custom instance configuration commands
```

### 3. Visual Studio Project Setup

#### 3.1 Project Creation
```powershell
# Create new ASP.NET Core MVC Project
dotnet new mvc -n ADLDSManagementPortal
cd ADLDSManagementPortal

# Install Required Packages
dotnet add package System.DirectoryServices.Protocols
dotnet add package Microsoft.AspNetCore.Identity
```

#### 3.2 LDAP Connection Service
```csharp
public class LdapConnectionService
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<LdapConnectionService> _logger;

    public LdapConnectionService(IConfiguration configuration, ILogger<LdapConnectionService> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }

    public LdapConnection CreateSecureConnection()
    {
        try 
        {
            var ldapServer = _configuration["LdapSettings:Server"];
            var bindDN = _configuration["LdapSettings:BindDN"];
            var bindPassword = _configuration["LdapSettings:BindPassword"];

            var credentials = new NetworkCredential(bindDN, bindPassword);
            var connection = new LdapConnection(new LdapDirectoryIdentifier(ldapServer, 389));
            
            connection.Credential = credentials;
            connection.AuthType = AuthType.Basic;
            
            // Optional: Configure SSL
            connection.SessionOptions.ProtocolVersion = 3;
            connection.SessionOptions.SecureSocketLayer = true;

            return connection;
        }
        catch (Exception ex)
        {
            _logger.LogError($"LDAP Connection Error: {ex.Message}");
            throw;
        }
    }
}
```

### 4. Security Implementations

#### 4.1 Authentication Middleware
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
        .AddCookie(options => 
        {
            options.LoginPath = "/Account/Login";
            options.AccessDeniedPath = "/Account/AccessDenied";
            options.ExpireTimeSpan = TimeSpan.FromHours(2);
        });

    services.AddAuthorization(options =>
    {
        options.AddPolicy("AdminOnly", 
            policy => policy.RequireRole("Administrator"));
    });
}
```

### 5. Deployment Strategies

#### 5.1 Local IIS Deployment
1. Publish ASP.NET Core Application
2. Configure IIS Application Pool
3. Set appropriate file permissions

#### 5.2 Docker Containerization
```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80 389 636

FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["ADLDSManagementPortal.csproj", "./"]
RUN dotnet restore "ADLDSManagementPortal.csproj"
COPY . .
RUN dotnet build "ADLDSManagementPortal.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "ADLDSManagementPortal.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "ADLDSManagementPortal.dll"]
```
If you're running both the Windows Server (as a VM) and the .NET application on the same host, Docker is not necessary. In this local development and testing scenario, you can directly deploy the ASP.NET Core application using:

IIS (Internet Information Services)
Kestrel (ASP.NET Core's built-in web server)

Benefits of Local Deployment without Docker:

Simpler setup
Direct communication between application and AD LDS
Lower overhead
Easier debugging
No containerization complexity

Recommended Local Deployment Steps:

Install IIS on Windows Server VM
Install .NET Core Hosting Bundle
Publish ASP.NET Core application directly to IIS
Configure application bindings and connection strings to point to local AD LDS instance
## 🛡️ Security Recommendations
1. Use LDAPS with SSL/TLS
2. Implement strong password policies
3. Use least-privilege access model
4. Regular security audits
5. Enable advanced logging

## 🔍 Troubleshooting
- **Connection Issues**: 
  - Verify firewall rules
  - Check LDAP server configuration
  - Validate credentials
- **Performance Concerns**:
  - Monitor server resources
  - Optimize LDAP queries
  - Implement caching mechanisms

## 📋 Compliance & Standards
- LDAP v3 Compliant
- OWASP Security Recommendations
- NIST Authentication Guidelines

## 🚧 Roadmap
- [ ] Implement Multi-Factor Authentication
- [ ] Create Comprehensive Logging
- [ ] Develop Advanced Reporting
- [ ] Containerize Complete Solution





**Final Note**: This project provides a robust framework for AD LDS management. Customize and extend based on specific organizational requirements.
