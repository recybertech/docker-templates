
# RHP Docker Templates

To begin with, you will need to install docker and docker-compose on your system by following the instructions provided in https://docs.docker.com/get-docker/.

## Docker Commands

1. Tag local image
    ```python
    #tag image
    docker tag <image-id> <repo-id>

    #example
    #docker tag 26b5c1deea9f recybertech/php-mssql:8.3
    ```

2. Push to repo
    ```python
    docker push <repo-id>

    #example
    #docker push recybertech/php-mssql:8.3
    ```
