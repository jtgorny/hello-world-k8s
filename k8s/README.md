# running locally
go run hello.go
http://localhost:8080/api/hello

# building docker image
docker build -t hello-world .

# tagging docker image
docker tag a23196d090a9 gornyj/hello-world:test

# pushing docker image
docker push gornyj/hello-world:test