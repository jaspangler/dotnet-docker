# escape=`

ARG REPO=mcr.microsoft.com/dotnet/aspnet

# Installer image
FROM mcr.microsoft.com/windows/servercore:ltsc2022-amd64 AS installer

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

RUN `
    # Retrieve .NET SDK
    Invoke-WebRequest -OutFile dotnet.zip https://dotnetcli.azureedge.net/dotnet/Sdk/5.0.404/dotnet-sdk-5.0.404-win-x64.zip; `
    $dotnet_sha512 = 'a6d254a46e93a41bf41df34c941503cfc5f61af20ffc0abc571bbaf238fd66f0fcc879e7181e1e1af788e96912b31012e817bf1202e55b8f27c17352f3f5528d'; `
    if ((Get-FileHash dotnet.zip -Algorithm sha512).Hash -ne $dotnet_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    mkdir dotnet; `
    tar -C dotnet -oxzf dotnet.zip; `
    Remove-Item -Force dotnet.zip; `
    `
    # Install PowerShell global tool
    $powershell_version = '7.1.5'; `
    Invoke-WebRequest -OutFile PowerShell.Windows.x64.$powershell_version.nupkg https://pwshtool.blob.core.windows.net/tool/$powershell_version/PowerShell.Windows.x64.$powershell_version.nupkg; `
    $powershell_sha512 = 'b654ad28e27382a1852fc64ddbed23ea1a2d9ffe71d48869cd911864589950346cf9a4603f635765962c28fab663e440ccd1bed235c2176b1caaed50e3c1e086'; `
    if ((Get-FileHash PowerShell.Windows.x64.$powershell_version.nupkg -Algorithm sha512).Hash -ne $powershell_sha512) { `
        Write-Host 'CHECKSUM VERIFICATION FAILED!'; `
        exit 1; `
    }; `
    \dotnet\dotnet tool install --add-source . --tool-path \powershell --version $powershell_version PowerShell.Windows.x64; `
    \dotnet\dotnet nuget locals all --clear; `
    Remove-Item -Force PowerShell.Windows.x64.$powershell_version.nupkg; `
    Remove-Item -Path \powershell\.store\powershell.windows.x64\$powershell_version\powershell.windows.x64\$powershell_version\powershell.windows.x64.$powershell_version.nupkg -Force; `
    `
    # Delete everything in the dotnet folder that's not needed in the SDK layer but will instead be derived from base layers
    Get-ChildItem -Exclude 'LICENSE.txt','ThirdPartyNotices.txt','packs','sdk','templates','shared' -Path dotnet `
        | Remove-Item -Force -Recurse; `
    Get-ChildItem -Exclude Microsoft.WindowsDesktop.App -Path dotnet\shared | Remove-Item -Force -Recurse

# SDK image
FROM $REPO:5.0.13-nanoserver-ltsc2022

ENV `
    # Unset ASPNETCORE_URLS from aspnet base image
    ASPNETCORE_URLS= `
    # Do not generate certificate
    DOTNET_GENERATE_ASPNET_CERTIFICATE=false `
    # SDK version
    DOTNET_SDK_VERSION=5.0.404 `
    # Enable correct mode for dotnet watch (only mode supported in a container)
    DOTNET_USE_POLLING_FILE_WATCHER=true `
    # Skip extraction of XML docs - generally not useful within an image/container - helps performance
    NUGET_XMLDOC_MODE=skip `
    # PowerShell telemetry for docker image usage
    POWERSHELL_DISTRIBUTION_CHANNEL=PSDocker-DotnetSDK-NanoServer-ltsc2022

# In order to set system PATH, ContainerAdministrator must be used
USER ContainerAdministrator
RUN setx /M PATH "%PATH%;C:\Program Files\powershell"
USER ContainerUser

COPY --from=installer ["/dotnet", "/Program Files/dotnet"]

COPY --from=installer ["/powershell", "/Program Files/powershell"]

# Trigger first run experience by running arbitrary cmd
RUN dotnet help
