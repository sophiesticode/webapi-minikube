# Stage 1: Build the application
FROM golang:1.21.5 AS builder
WORKDIR /app
COPY . .
RUN go mod tidy
RUN go build -o webapi .

# Stage 2: Run the application
FROM ubuntu:22.04
COPY --from=builder /app/webapi .
EXPOSE 8081
CMD ["./webapi"]
