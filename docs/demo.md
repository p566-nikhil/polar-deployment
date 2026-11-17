## 1. Check Kubernetes Services
Command:
kubectl get pods

Result:
All five services were running:
- catalog-service: 1/1 Running
- edge-service: 1/1 Running
- polar-keycloak: 1/1 Running
- polar-postgres: 1/1 Running
- polar-redis: 1/1 Running

## 2. Test Public Book Browsing
Command:
curl -i http://localhost:9000/books

Result:
HTTP 200 OK. Books were returned through Edge Service without login.

## 3. Test Isabelle Login
Browser:
http://localhost:9000/oauth2/authorization/keycloak

Credentials:
- Username: isabelle
- Password: password

Command:
curl http://localhost:9000/user

Result:
{
  "username": "isabelle",
  "firstName": "Isabelle",
  "lastName": "Dahl",
  "roles": ["employee", "customer"]
}

## 4. Test Bjorn Authorization
Command:
curl -i -X POST http://localhost:9001/books \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "isbn": "2222222223",
    "title": "Customer Book",
    "author": "Polar",
    "price": 10,
    "publisher": "Polarsophia"
  }'


Result:
HTTP 403 Forbidden. Bjorn could not create a book because he only has the customer role.

## 5. Test Anonymous Access
Command:
curl -i -X POST http://localhost:9001/books \
  -H "Content-Type: application/json" \
  -d '{
    "isbn": "3333333333",
    "title": "Anonymous Book",
    "author": "Polar",
    "price": 10,
    "publisher": "Polarsophia"
  }'


Result:
HTTP 401 Unauthorized. Anonymous users cannot create books.

## 6. Delete the Edge Service Pod while logged in: after the replacement Pod is ready, /user still knows you. Explain why

/user still knows you because access tokens are stored in Redis for short term memory instead of being tied to each Pod. So when a Pod is deleted, the replacement can retrieve data from Redis.