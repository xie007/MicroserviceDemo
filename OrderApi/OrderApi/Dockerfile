FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["OrderApi/OrderApi.csproj", "OrderApi/"]
COPY ["OrderApi.Data/OrderApi.Data.csproj", "OrderApi.Data/"]
COPY ["OrderApi.Domain/OrderApi.Domain.csproj", "OrderApi.Domain/"]
COPY ["OrderApi.Service/OrderApi.Service.csproj", "OrderApi.Service/"]
COPY ["OrderApi.Messaging.Receive/OrderApi.Messaging.Receive.csproj", "OrderApi.Messaging.Receive/"]
COPY ["Tests/OrderApi.Test/OrderApi.Test.csproj", "Tests/OrderApi.Test/"]  
COPY ["Tests/OrderApi.Service.Test/OrderApi.Service.Test.csproj", "Tests/OrderApi.Service.Test/"]  
COPY ["Tests/OrderApi.Data.Test/OrderApi.Data.Test.csproj", "Tests/OrderApi.Data.Test/"] 
COPY ["*.props", "./"]

RUN dotnet restore "OrderApi/OrderApi.csproj"
RUN dotnet restore "Tests/OrderApi.Test/OrderApi.Test.csproj"
RUN dotnet restore "Tests/OrderApi.Service.Test/OrderApi.Service.Test.csproj"
RUN dotnet restore "Tests/OrderApi.Data.Test/OrderApi.Data.Test.csproj"
COPY . .

RUN dotnet build "OrderApi/OrderApi.csproj" -c Release -o /app/build --no-restore
RUN dotnet build "Tests/OrderApi.Test/OrderApi.Test.csproj" -c Release --no-restore
RUN dotnet build "Tests/OrderApi.Service.Test/OrderApi.Service.Test.csproj" -c Release --no-restore
RUN dotnet build "Tests/OrderApi.Data.Test/OrderApi.Data.Test.csproj" -c Release --no-restore

FROM build AS test  
ARG BuildId
LABEL test=${BuildId} 
RUN dotnet test --no-build -c Release --results-directory /testresults --logger "trx;LogFileName=test_results.trx"  Tests/OrderApi.Test/OrderApi.Test.csproj  
RUN dotnet test --no-build -c Release --results-directory /testresults --logger "trx;LogFileName=test_results2.trx" Tests/OrderApi.Service.Test/OrderApi.Service.Test.csproj  
RUN dotnet test --no-build -c Release --results-directory /testresults --logger "trx;LogFileName=test_results3.trx" Tests/OrderApi.Data.Test/OrderApi.Data.Test.csproj

FROM build AS publish
RUN dotnet publish "OrderApi/OrderApi.csproj" --no-restore -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "OrderApi.dll"]