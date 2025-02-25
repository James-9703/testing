
This project is a User Activity Monitoring (UAM) system built using Apache Kafka, PostgreSQL, ksqlDB, and Docker. It tracks user activities, monitors sudo commands, detects security violations, and logs events for auditing purposes. The system leverages Kafka for real-time data streaming, ksqlDB for stream processing, PostgreSQL for persistent storage, and Prometheus for monitoring.
Features

    Real-time User Activity Tracking: A producer sends user activity data to a Kafka broker at fixed intervals.
    Sudo Command Monitoring: A dedicated producer watches for sudo commands and forwards them to the Kafka broker.
    Security Violation Detection: A Kafka Streams application identifies potential security violations from user activity.
    Audit Trail: Sudo commands are logged into a PostgreSQL database for auditing.
    Monitoring: Prometheus tracks the Kafka broker's online status.
    Containerized Deployment: All components (Kafka broker, PostgreSQL, Prometheus, ksqlDB, and ksql CLI) are managed via Docker Compose.

Prerequisites
Before setting up the system, ensure you have the following installed:

    Git
    Docker and Docker Compose
    Java (for running the producer and sudo watcher applications)
    A Unix-like environment (e.g., Linux or macOS) with sudo command support

Setup Instructions

    Clone the Repository
    bash

    git clone <repository-url>
    cd UAM/

    Install Dependencies
    Run the setup script to install necessary tools and configure the environment:
    bash

    sudo ./setup.sh

    Start Core Services
    Launch Kafka broker, PostgreSQL, Prometheus, ksqlDB, and ksql CLI using Docker Compose:
    bash

    docker compose -f kafka.yml up

Usage
1. Monitor with Prometheus
Prometheus is used to monitor the Kafka broker's status. After starting the core services:

    Open your browser and navigate to http://localhost:9090/.
    Run the query up to verify that the Kafka broker instance is online.

2. Start the User Activity Producer
This application generates and sends user activity data to the Kafka broker:
bash

cd producer/app/build/libs/
java -jar app.jar

3. Start the Sudo Command Watcher
This application monitors sudo commands and sends them to the Kafka broker:
bash

cd ~/UAM/sudoWatch/app/build/libs/
java -jar app.jar

4. Test with a Sample Sudo Command
Run a sudo command to produce sample data:
bash

sudo ls

5. Start Stream Processing
Launch the Kafka Streams applications to process data for security violations and audit logging:
bash

cd ~/UAM/
docker compose -f process.yml up

6. Verify Sudo Command Logging
Check if the sudo command is logged in the PostgreSQL database:
bash

docker exec -it db psql -U postgres

In the PostgreSQL shell, run:
sql

select * from sudo;

7. Monitor Security Violations
Start the ksql CLI as a dashboard to query the Kafka stream:
bash

docker exec -it ksqldb-cli ksql http://ksqldb-server:8088

Create a stream to read user activity from the Kafka topic statusV:
sql

CREATE STREAM violation (
    username STRING, 
    ip STRING, 
    openedApp ARRAY<VARCHAR>, 
    idleTime INT, 
    firewall BOOLEAN, 
    pw INT, 
    violation STRING
) WITH (
    kafka_topic='statusV', 
    value_format='json'
);

Query the stream to display violations in real-time:
sql

SELECT username, ip, violation FROM violation EMIT CHANGES;

Project Structure

    kafka.yml: Docker Compose file for core services (Kafka, PostgreSQL, Prometheus, ksqlDB, etc.).
    process.yml: Docker Compose file for Kafka Streams applications.
    producer/app/: User activity producer application.
    sudoWatch/app/: Sudo command watcher application.
    setup.sh: Setup script for dependencies and environment configuration.

Notes

    Ensure all Docker containers are running properly before starting the producer or stream processing applications.
    The system assumes the Kafka topic statusV is used for user activity data. Adjust configurations if your topic name differs.
    For production use, consider securing PostgreSQL with a password and restricting Docker container access.

Contributing
Feel free to submit issues or pull requests to improve this project. Contributions are welcome!
License
This project is licensed under the MIT License (LICENSE).
