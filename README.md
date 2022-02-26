# mental_changelog
Miscellaneous notes about things I encounter while working.


## Getting Started

### Prerequisites
 You must have Docker and Compose installed to run your Jekyll project in Docker.

 [Instructions to install Docker Compose](https://docs.docker.com/compose/install/)

### Testing Locally

To serve up the project:
1) In project folder run:
```
sudo docker-compose up
```
2) View project at:
```
localhost:4000
```
3) Edit project and Jekyll will auto-regenerates the site
4) When you want to shut-down the site, use Ctrl+C then run:
```
docker-compose down
