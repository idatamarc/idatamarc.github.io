## What and how should we promisify in your NodeJS layer


#### Promisify your Redis communications

**Decoration with `bluebird`**
```
const redisClient       = (require('redis')).createClient(
  process.env.REDIS_PORT,
  process.env.REDIS_HOSTNAME,
  {
      auth_pass: process.env.REDIS_PASSWORD
      //tls: process.env.REDIS_PASSWORD && {servername: process.env.REDIS_HOSTNAME}
});

const client = require('bluebird').promisifyAll(redisClient);
```
* Sample code for `get` your express session data from Redis:
    ```
    getSession = async (sessionId) => {
    
        let session = await client.getAsync('sess:'+sessionId);
        session = JSON.parse(session)
    
        return session;
    }
    ```
* Sample code for `get` your express session data from Redis (with chaining):
    ```
    getSession = async (sessionId) => {
    
       return client.getAsync('sess:'+sessionId)
              .then((session, err) => err ? Promise.reject(`on ${sessionId}: ${err}`) : Promise.resolve(JSON.parse(session))) 
    }
    ```    
* Sample code for `set` your express session data from Redis:
    ```
    createSession = async (sessionId) => {
    
        let mySession = {"dummy_field1": "dummy_value1", "dummy_field2": "dummy_value2"} ;
        
        return client.setAsync('sess:'+sessionId, JSON.stringify(mySession))
              .then((res, err) => err ? Promise.reject(`on ${sessionId}: ${err}`) : Promise.resolve(res))
    }
    ```    

**Decoration with `util`**
```
const redisClient       = (require('redis')).createClient(
  process.env.REDIS_PORT,
  process.env.REDIS_HOSTNAME,
  {
      auth_pass: process.env.REDIS_PASSWORD
      //tls: process.env.REDIS_PASSWORD && {servername: process.env.REDIS_HOSTNAME}
  }
);
const { promisify } = require("util");
const getAsync = promisify(client.get).bind(redisClient);
const setAsync = promisify(client.set).bind(redisClient);
```
* Sample code for `get` your express session data from Redis:
    ```
    getSession = async (sessionId) => {
    
        let session = await getAsync('sess:'+sessionId);
        session = JSON.parse(session)
    
        return session;
    }
    ```
* Sample code for `get` your express session data from Redis: 
    ```
    getSession = async (sessionId) => {
    
       return getAsync('sess:'+sessionId)
              .then((session, err) => err ? Promise.reject(`on ${sessionId}: ${err}`) : Promise.resolve(JSON.parse(session))) 
    }
    ```    
* Sample code for `set` your express session data from Redis:
    ```
    createSession = async (sessionId) => {
        
        let mySession = {"dummy_field1": "dummy_value1", "dummy_field2": "dummy_value2"} ;
            
        return setAsync('sess:'+sessionId, JSON.stringify(mySession))
               .then((res, err) => err ? Promise.reject(`on ${sessionId}: ${err}`) : Promise.resolve(res))
    }
    ```   
**Notes**: If you are looking for a easier way to promisify `all` redis communication with util, you might want to refer to neat library [redis-promisify](https://github.com/zenxds/redis-promisify), and below is the essential code snippet:
```
const util = require('util')
const redis = require("redis")
const redisCommands = require('redis-commands')

promisify(redis.RedisClient.prototype, redisCommands.list)
promisify(redis.Multi.prototype, ['exec', 'execAtomic'])

function promisify(obj, methods) {
  methods.forEach((method) => {
    if (obj[method])
      obj[method + 'Async'] = util.promisify(obj[method])
  })
}

module.exports = redis
```

**Decorate with `async-redis`**
```
const redisClient       = (require('redis')).createClient(
  process.env.REDIS_PORT,
  process.env.REDIS_HOSTNAME,
  {
      auth_pass: process.env.REDIS_PASSWORD
      //tls: process.env.REDIS_PASSWORD && {servername: process.env.REDIS_HOSTNAME}
  }
);

const client = require("async-redis").decorate(redisClient);
client.on("error", function (err) {
  log.error("Error in connecting to Redis" + err);
});
```
* Sample code for `get` your express session data from Redis:
    ```
    getSession = async (sessionId) => {
    
        let session = await client.get('sess:'+sessionId);
        session = JSON.parse(session)
    
        return session;
    }
    ```
* Sample code for `set` your express session data from Redis:
    ```
    createSession = async (sessionId) => {
        
        let mySession = {"dummy_field1": "dummy_value1", "dummy_field2": "dummy_value2"} ;
            
        await client.set('sess:'+sessionId, JSON.stringify(mySession));
    }
    ``` 

**Caution** : Please adopt one of the above in your project, otherwise it might bring some unexpected complications. I have personally tried mixing `async-redis` and `bluebird` to promisify redis comms, which leads to scenarios like `only native reids client works`. 

#### Promisify your MySQL communications
```
const mysqlConn = mysql.createConnection({
                  host: 'yourMySQLHost',
                  user: 'yourUserName',
                  password: 'yourPassword',
                  database: 'yourDB'
          });
const promiseMysqlQuery = promisify(mysqlConn.query).bind(mysqlConn);

async function fetchDataFromGate3Mysql(promiseMysqlQuery, sqlStatement) {

    //console.log(sqlStatement);
    let jsonResultSet = [];

    await promiseMysqlQuery({sql: sqlStatement}) //{sql: 'SELECT COUNT(*) AS count FROM big_table', timeout: 60000}
        .then((results, fields) => {

            //slice the array with 30k incr
            let from = 0;
            let jsonResultSlice = [];
            do {
                //reset
                jsonResultSlice = [];
                let stringResultSlice = JSON.stringify(results.slice(from, from+30000));
                jsonResultSlice = JSON.parse(stringResultSlice);

                jsonResultSet = jsonResultSet.concat(jsonResultSlice);
                from+=30000;
            }
            while (jsonResultSlice.length > 0);

        })
        .catch(err => {
            console.error(err)
        });

    console.log('>> length of result: ', jsonResultSet.length);
    return jsonResultSet;
}
```


### What about your MongoDB communications
