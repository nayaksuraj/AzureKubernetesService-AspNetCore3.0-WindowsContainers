# escape=`
FROM mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2019 as base

## Install dotnet 3.0.0-preview5 hosting pack
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'Continue'; $verbosePreference='Continue';"]
ADD https://download.visualstudio.microsoft.com/download/pr/41483896-e029-40ff-b167-b4dde24153e1/ccd755c56bc8cfeb38ef816e54deb906/dotnet-hosting-3.0.0-preview5-19227-01-win.exe "C:/setup/dotnet-hosting-3.0.0-preview5-19227-01-win.exe"
RUN start-process -Filepath "C:/setup/dotnet-hosting-3.0.0-preview5-19227-01-win.exe" -ArgumentList @('/install', '/quiet', '/norestart') -Wait 
RUN Remove-Item -Force "C:/setup/dotnet-hosting-3.0.0-preview5-19227-01-win.exe"

FROM mcr.microsoft.com/dotnet/core/sdk:3.0 AS build
WORKDIR /src
COPY ["SampleWebApp.csproj", "SampleWebApp/"]

RUN dotnet restore "SampleWebApp/SampleWebApp.csproj"
COPY . SampleWebApp/.
WORKDIR "/src/SampleWebApp"
RUN dotnet build "SampleWebApp/SampleWebApp.csproj" -c Release -o /app

FROM build AS publish
RUN dotnet publish "SampleWebApp/SampleWebApp.csproj" -c Release -o /app

FROM base AS final
WORKDIR /inetpub/wwwroot/SampleWebApp

RUN Remove-Website -Name 'Default Web Site';

RUN Import-Module WebAdministration; `
    New-WebAppPool -Name 'ap-SampleWebApp'; `
    Set-ItemProperty IIS:\AppPools\ap-SampleWebApp -Name managedRuntimeVersion -Value ''; `
    Set-ItemProperty IIS:\AppPools\ap-SampleWebApp -Name enable32BitAppOnWin64 -Value 0; `
    Set-ItemProperty -Path IIS:\AppPools\ap-SampleWebApp -Name processModel -value @{identitytype='LocalSystem'}

RUN New-Website -Name 'SampleWebApp' `
                -Port 80 -PhysicalPath 'C:\inetpub\wwwroot\SampleWebApp' `
                -ApplicationPool 'ap-SampleWebApp' -force

COPY --from=publish /app .

EXPOSE 80
