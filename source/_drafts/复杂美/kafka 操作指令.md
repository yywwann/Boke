kafka-topics.sh --bootstrap-server localhost:9092 --topic goim-dtalk-topic --alter --partitions 16; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic offpush-dtalk-topic --alter --partitions 16; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic received-dtalk-topic --alter --partitions 16; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic store-dtalk-topic --alter --partitions 16

kafka-topics.sh --bootstrap-server localhost:9092 --topic goim-dtalk-topic  --describe; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic offpush-dtalk-topic  --describe; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic received-dtalk-topic  --describe; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic store-dtalk-topic  --describe

kafka-topics.sh --bootstrap-server localhost:9092 --topic goim-dtalk-topic  --delete; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic offpush-dtalk-topic  --delete; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic received-dtalk-topic  --delete; \
kafka-topics.sh --bootstrap-server localhost:9092 --topic store-dtalk-topic  --delete

kafka-topics.sh --bootstrap-server localhost:9092 --topic test-topic  --describe
kafka-topics.sh --bootstrap-server localhost:9092 --topic test-topic --alter --partitions 2

sudo supervisorctl stop all; \
sudo supervisorctl restart chain33 backend disc backup generator oss user offline-push; \
sudo supervisorctl restart comet logic; \
sudo supervisorctl restart answer pusher group store call; \
sudo supervisorctl status



$ kafka-topics.sh --bootstrap-server localhost:9092 --topic store-dtalk-topic --create --replication-factor 1 --partitions 1024

# 查看所有 topic
$ kafka-topics.sh --bootstrap-server localhost:9092 --list
