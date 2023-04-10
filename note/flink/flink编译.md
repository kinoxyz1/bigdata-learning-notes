




---
# clone code
git clone https://github.com/apache/flink.git

# checkout branch
```bash
git checkout release-1.12
```

# build code
```bash
mvn install -Dmaven.test.skip=true -Drat.skip=true -Dmaven.javadoc.skip=true -Dcheckstyle.skip=true -Dfast -T 4 -Dmaven.compile.fork=true
```