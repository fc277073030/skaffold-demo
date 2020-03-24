FROM golang:1.13.9-alpine3.11 as builder
COPY app.go .
RUN go build -o /app .

FROM alpine:3.11
COPY --from=builder /app .
CMD ["./app"]
