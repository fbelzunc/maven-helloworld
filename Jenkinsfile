stage 'first'
node {
    def delay = new Random().nextInt(60)
    echo "Sleeping for ${delay}s..."
    sleep time: delay, unit: 'SECONDS'
    checkout scm
}
stage concurrency: 1, name: 'second'
node {
    def delay = new Random().nextInt(60)
    echo "Sleeping for ${delay}s..."
    sleep time: delay, unit: 'SECONDS'
}

