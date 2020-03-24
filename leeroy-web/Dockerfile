FROM golang:1.13.9-alpine3.11 as builder
COPY web.go .
RUN go build -o /web .

FROM alpine:3.11
COPY --from=builder /web .
CMD ["./web"]