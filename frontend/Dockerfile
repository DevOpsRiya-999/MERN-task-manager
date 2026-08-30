FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build


# -------------------------runner -------------------------


FROM nginx:1.27-alpine


COPY nginx.conf /etc/nginx/conf.d/default.conf

COPY --from=builder /app/build /usr/share/nginx/html

Expose 80

cmd ["nginx", "-g"," daemon off;"]
