FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY ["Yale.Portal/Yale.Portal.csproj", "Yale.Portal/"]
RUN dotnet restore "Yale.Portal/Yale.Portal.csproj"
COPY . .
WORKDIR "/src/Yale.Portal"
RUN dotnet build "Yale.Portal.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Yale.Portal.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Yale.Portal.dll"]