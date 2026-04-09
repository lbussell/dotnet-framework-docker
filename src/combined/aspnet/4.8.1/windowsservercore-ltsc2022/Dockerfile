# escape=`

FROM mcr.microsoft.com/windows/servercore:ltsc2022-amd64

ENV `
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true `
    COMPLUS_RUNNING_IN_CONTAINER=1 `
    COMPLUS_NGenProtectedProcess_FeatureEnabled=0

RUN `
    # Install .NET Fx 4.8.1
    curl -fSLo dotnet-framework-installer.exe https://download.visualstudio.microsoft.com/download/pr/6f083c7e-bd40-44d4-9e3f-ffba71ec8b09/3951fd5af6098f2c7e8ff5c331a0679c/ndp481-x86-x64-allos-enu.exe `
    && .\dotnet-framework-installer.exe /q `
    && del .\dotnet-framework-installer.exe `
    && powershell Remove-Item -Force -Recurse ${Env:TEMP}\* `
    `
    # Apply latest patch
    && curl -fSLo patch.msu https://catalog.s.download.windowsupdate.com/c/msdownload/update/software/secu/2025/09/windows10.0-kb5066134-x64-ndp481_103b0d4cef1c6b42632f1cb043725ded0fce0ed3.msu `
    && mkdir patch `
    && expand patch.msu patch -F:* `
    && del /F /Q patch.msu `
    && dism /Online /Quiet /Add-Package /PackagePath:C:\patch\windows10.0-KB5066134-x64-ndp481.cab `
    && rmdir /S /Q patch `
    `
    # Ngen top of assembly graph to optimize a set of frequently used assemblies
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen install "Microsoft.PowerShell.Utility.Activities, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" `
    # To optimize 32-bit assemblies, uncomment the next line
    # && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen install "Microsoft.PowerShell.Utility.Activities, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen update `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen update

RUN dism /Online /Quiet /Enable-Feature /All /FeatureName:IIS-WebServerRole /FeatureName:NetFx4Extended-ASPNET45 /FeatureName:IIS-ASPNET45 `
    && dism /Online /Quiet /Disable-Feature /FeatureName:IIS-WebServerManagementTools `
    && del /q "C:\inetpub\wwwroot\*" `
    && for /D %p IN ("C:\inetpub\wwwroot\*") DO rmdir "%p" /s /q `
    && curl -fSLo ServiceMonitor.exe https://github.com/microsoft/IIS.ServiceMonitor/releases/download/v2.0.1.10/ServiceMonitor.exe `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen update `
    && %windir%\Microsoft.NET\Framework\v4.0.30319\ngen update

# Install 2.9.0 Roslyn compilers
RUN curl -fSLo microsoft.net.compilers.2.9.0.zip https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet-public/nuget/v3/flat2/microsoft.net.compilers/2.9.0/microsoft.net.compilers.2.9.0.nupkg `
    && mkdir C:\RoslynCompilers `
    && tar -C C:\RoslynCompilers -zxf microsoft.net.compilers.2.9.0.zip `
    && del microsoft.net.compilers.2.9.0.zip `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen install C:\RoslynCompilers\tools\csc.exe /ExeConfig:C:\RoslynCompilers\tools\csc.exe `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen install C:\RoslynCompilers\tools\vbc.exe /ExeConfig:C:\RoslynCompilers\tools\vbc.exe `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen install C:\RoslynCompilers\tools\VBCSCompiler.exe /ExeConfig:C:\RoslynCompilers\tools\VBCSCompiler.exe

# Install 3.6.0 Roslyn compilers
RUN curl -fSLo microsoft.net.compilers.3.6.0.zip https://pkgs.dev.azure.com/dnceng/public/_packaging/dotnet-public/nuget/v3/flat2/microsoft.net.compilers/3.6.0/microsoft.net.compilers.3.6.0.nupkg `
    && mkdir C:\RoslynCompilers-3.6.0 `
    && tar -C C:\RoslynCompilers-3.6.0 -zxf microsoft.net.compilers.3.6.0.zip `
    && del microsoft.net.compilers.3.6.0.zip `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen install C:\RoslynCompilers-3.6.0\tools\csc.exe /ExeConfig:C:\RoslynCompilers-3.6.0\tools\csc.exe `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen install C:\RoslynCompilers-3.6.0\tools\vbc.exe /ExeConfig:C:\RoslynCompilers-3.6.0\tools\vbc.exe `
    && %windir%\Microsoft.NET\Framework64\v4.0.30319\ngen install C:\RoslynCompilers-3.6.0\tools\VBCSCompiler.exe /ExeConfig:C:\RoslynCompilers-3.6.0\tools\VBCSCompiler.exe

ENV ROSLYN_COMPILER_LOCATION=C:\RoslynCompilers-3.6.0\tools

EXPOSE 80

ENTRYPOINT ["C:\\ServiceMonitor.exe", "w3svc"]
