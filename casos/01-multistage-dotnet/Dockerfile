FROM mcr.microsoft.com/dotnet/sdk:8.0 AS Build
WORKDIR /src 

COPY app/*.csproj ./
RUN dotnet restore

COPY app/ ./
RUN dotnet publish -c Release -o /app/publish --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS Runtime
WORKDIR /app
COPY --from=Build /app/publish .

EXPOSE 8080
ENTRYPOINT ["dotnet", "docker-study-api.dll"]