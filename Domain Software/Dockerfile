# TODO, better add docker build with multipe inherited dockerfiles
FROM node:carbon-alpine AS frontend-build

# Upgrade OS dependencies
RUN apk update && apk upgrade

# Set the working directory to the location of our app
WORKDIR /opt/app

#Timezone
RUN apk add tzdata && cp /usr/share/zoneinfo/UTC /etc/localtime
RUN echo "UTC" > /etc/timezone
RUN date && apk del tzdata

# Copy Node.js App dependency manifest
COPY ionic-app-skeleton/package.json ./package.json
# Install Node.js dependencies
RUN npm install
# RUN mkdir -p /opt/app && cp -a /tmp/node_modules /opt/app/
# Copy our app from the hard drive to the working direcotry in the docker image
COPY ionic-app-skeleton/ ./
# Build the App inside the image
RUN npm run build --prod

FROM microsoft/dotnet:2.1-sdk AS backend-build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY dotnet-api-backend-skeleton/*.csproj ./
RUN dotnet restore

# copy everything else and build
COPY dotnet-api-backend-skeleton/ ./
# RUN dotnet build -c Release 
RUN dotnet publish -c Release -o out
COPY --from=frontend-build /opt/app/www ./out/wwwroot

##################
# Production image 
FROM microsoft/dotnet:2.1-runtime-alpine AS deploy-env
# use aspnet core https://github.com/NileshGule/dotnet-2017/blob/master/DotNet2017/CoreWebAPI/Dockerfile

# Upgrade to newest packages
RUN apk update && apk upgrade
# RUN apk add libc6-compat libunwind
#Timezone
RUN apk add tzdata && cp /usr/share/zoneinfo/UTC /etc/localtime
RUN echo "UTC" > /etc/timezone
RUN date && apk del tzdata

# Default port for this App Skeleton, will be exposed when -P specified
ENV ASPNETCORE_URLS=http://+:5050
# EXPOSE 80
EXPOSE 5050

# Copy from build stage
WORKDIR /app
COPY --from=backend-build /app/out ./

# using array notation causes node to be PID1 and will not exit properly. Without the array notation the shell forwards the sigterm correctly. 
ENTRYPOINT ["dotnet", "dotnet-api-backend-skeleton.dll"]
