# 详细说明如何在大型项目中使用TypeOrm 进行数据库管理，及其性能和维护的影响?
typeorm是一个支持typescript的orm框架，可以大幅提高数据库操作的开啊效率，但在大型项目中使用时需要注意性能和维护的影响
1. 数据库表实体定义：
- 通过typeorm的装饰器（如@Entity，@Column等）来定义数据库的实体结构，这种方式便于开发者理解数据模型，但需要注意实体定义的复杂度
- 在大型项目中，建议将实体与业务逻辑分开，以便于代码的模块化管理
2. 查询优化:
- lazy relations 和eager relations: 使用lazy relations可以减少不必要的关联查询，适合需要按需加载关联数据的场景。而eager relations可以简化需要频繁关联查询的数据操作
- queryBuilder：typeorm提供了queryVBuilder以支持复杂查询场景，通过queryBuilder可以对查询进行更精细的的控制，并进行性能优化
3. 数据库迁移管理：
- 使用Typeorm的migration功能管理数据库表的变更，特别适合多开发者并行开发的场景，可以确保数据库的版本一致
- 在大型项目中，数据库变更频繁，使用migration可以自动化变更管理，但需要确保迁移脚本的可靠性和测试
4. 性能影响：
- 懒加载（lazy loading）:在有大量关系表的情况下，懒加载可能会导致n+1查询问题，影响性能。应当通过性能监控工具（如APM）发现并优化
- 批量操作：尽量使用批量插入和更新操作，避免逐条处理，以提升数据库的操作效率

# redis的持久化机制对数据一致性有何影响？如何在生产环境中平衡redis的高性能与数据一致性?
redis提供两种持久化机制：RDB(redis database file)和 AOF（Append Only File）,他们各自的特性对数据一致性有不同的影响:
1. RDB（快照）持久化:
- 原理：定期将数据快照的形式保存到磁盘中。备份间隔可以配置，但通常为秒级或分钟级。
- 数据一致性影响：如果redis崩溃，可能会丢失最近一次快照后的所有数据。对于高频更新的数据，这种方式存在一定的数据丢失风险，但性能较好
2. AOF(追加文件)持久化:
- 原理：通过每次写入操作记录到日志中，redis可以在重启时重新执行日志中的命令以恢复数据
- 数据一致性影响：AOF具有更高的数据一致性保障（特别是设置为always或everysec模式时），但会对性能有一定的影响，特别是在高并发场景下，AOF会增加磁盘I/O压力
3. 在生产环境中平衡性能与一致性:
- 混合持久化：redis6.0及之后版本支持RDB和AOF混合持久化方式，可以减少AOF重写的时间并提供较好的数据一致性
- 复制和高可用：配置redis主从复制（master-slave），并结合redis sentinel实现高可用性。在主节点出现故障时，可以自动切换到从节点，以保证数据可用性和一致性,
- 合理配置持久化策略：在高性能要求的场景下，可以减少AOF的同步频率，如每秒持久化一次（everysec），以在性能和数据一致性之间取得平衡

# 说说Nest.js的依赖注入机制及其在复杂应用中的实现原理?
nest.js的依赖注入（dependency injection，DI）机制是其核心功能之一，用于管理模块、控制器、服务等组件之间的依赖关系.
- 依赖注入的工作原理:
  - 模块化结构：nest.js是模块化的，每个模块都会声明一组providers,这些提供者（providers）是服务或类的实例，它们可以在模块或全局范围内被注入使用
  - @Injectable装饰器：类通过@Injectable装饰器声明为可注入，Nest的IOC容器会自动处理类的实例化
  - 构造器注入：在需要注入依赖的类中，通过构造函数参数来标注所需的依赖。Nest会解析这些依赖，并根据模块的providers配置进行实例化和注入
  - 声明周期管理：nest.js 还可以通过scope来管理依赖的声明周期，如单例（Singleton）和请求（Request）作用域，以适应不同的场景需求
- 在复杂应用中的实现
  - 跨模块依赖：当一个模块依赖于另一个模块的服务时，必须将该服务导出(exports),以便其他模块乐意导入并使用
  - 动态模块： 在复杂的应用中，nest.js支持创建动态模块，允许模块根据传入的配置动态加载服务，处理复杂的配置需求

# nest.js中拦截器（interceptor）是如何实现的？请描述其设计原理和应用场景
拦截器（interceptor）是nest.js中用于在请求进入和响应离开的执行额外逻辑 的强大机制，类似于aop(面向切面编程)的设计理念
- 拦截器的设计原理：
   - 实现NestInterceptor接口：所有的拦截器都需要实现nest.js提供的NestInterceptor接口，该接口定义了intercept方法，用于接收请求处理逻辑的执行上下文（executionContext）和请求处理链（CallHandler）
   - 执行顺序：拦截器可以在方法调用前执行逻辑（如请求日志记录、用户身份验证等），也可以在方法调用后处理响应（如数据格式转换、性能监控）
   - 管道模式：拦截器的实现遵循类似中间件的管道模式，即可以在请求进行时和响应返回时插入自定义逻辑.
 - 应用场景：
   - 日志记录和监控：拦截器可以记录请求的响应时间、输入参数、输出结果等信息，用于系统监控和日志记录
   - 响应数据转换：拦截器可对返回的数据进行二次加工，如格式化响应数据，去除敏感信息等
   - 权限验证：可在进入控制器之前，检查用户权限，控制用户是否能访问指定的资源

# nest.js中装饰器（decorator）与元数据反射的实现原理是什么？如何通过自定义装饰器简化业务逻辑
装饰器（decorator）是nest.js基于typescript的元数据反射系统（Metadata refelction system）实现的，用于扩展类、方法或参数的行为。通过装饰器，我们可以在编译时为类添加元数据，从而在运行时读取并执行相应的逻辑
- 装饰器与元数据反射的实现原理
  - reflect-metadata库：nest.js使用typescript的reflect-metadata库来读取和定义元数据。通过装饰器，我们可以为类、方法、参数等附加元数据，nest会在应用运行时通过反射机制读取这些元数据，并基于此执行相应的逻辑
  - 内置装饰器：nest.js提供了多种内置装饰器，如@Controller、@Injecxtable、@Get等，用于声明控制器、服务或定义路由。每个装饰器都基于元数据反射系统工作
  - 自定有装饰器：我们可以通过@SetMetadata()或直接定义类或方法装饰器，来实现业务逻辑上的自定义行为
- 自定义装饰器的应用场景:
  - 权限控制：可以创建自定义装饰器，如@Roles(),通过为路由方法附加元数据，在请求处理过程中根据用户角色进行权限验证
  - 参数校验：自定义装饰器可以简化请求参数的验证逻辑。例如，使用@User()装饰器从JWT令牌中直接提取用户信息，而不是在每个控制器中手动解析.

# 说说如何在nestjs中集成postgresql数据库，解释其工作原理，并说说你对orm框架的理解?
在nestjs中集成postgresql和redis数据库可以通过typeorm和其他数据库驱动实现。以下是集成步骤及其工作原理:
- postgresql集成：
  - 使用typeorm作为orm工具，typeorm提供了对postgresql的支持，可以自动管理数据库的迁移、连接和查询
  - 安装typeorm和pg(postgresql驱动)
   ```bash
    npm install @nestjs/typeorm typeorm pg
   ```
  - 配置数据库连接
  在app.module.ts中配置typeorm模块
  ```bash
  TypeOrmModule.forRoot({
    type: 'postgres',
    host: 'localhost',
    port: 5432,
    username: 'postgres',
    password: 'password',
    database: 'test',
    entities: [__dirname + '/**/*.entity{.ts,.js}'],
    synchronize: true,
  })
  ```
  - 实现查询操作：通过repository对数据库进行增删改查操作
- 工作原理：
   - orm：全称obejct relation mapping/对象关系映射，对象关系应身（orm）是一种程序设计拘束，用于将不同类型的数据进行转换，实现面向对象编程语言对数据库之间的交互，它可以常见一个虚拟对象数据库，方便编程语言的使用.
   - typeorm:作为对象关系映射工具，typeorm提供了对postgresql的支持,自动管理数据库的连接池、sql语句的生成和事务处理

# 说说如何使用@nestjs/jwt 和@nestjs/passport 实现用jwt身份认证，并解释其原理?
jwt身份认证是通过在用户登录后生成一个加密的令牌（token），并在之后的请求中附带该令牌已证明用户身份。nestjs使用@nestjs/jwt和@nestjs/passport这两个库来实现jwt身份认证.具体实现步骤及其原理如下:
- 安装必要的包
  ```bash
  npm install @nestjs/jwt @nestjs/passport passport passport-jwt
  ```
- 配置jwt模块
  在auth.module.ts中配置jwt策略
  ```bash
      JwtModule.register({
        secret: 'secretKey',
        signOptions: { expiresIn: '60s' },
      })
  ```
- 实现jwt策略
  - 创建jwt策略类，通过passport-jwt来验证令牌
   ```bash
   @Injectable()
   export class JwtStrategy extends PassportStrategy(Strategy) {
     constructor() {
       super({
         jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
         ignoreExpiration: false,
         secretOrKey: 'secretKey',
       });
     }

     async validate(payload: any) {
       return { userId: payload.sub, username: payload.username };
     }
   }
   ```
- 用户认证与生成token
  - 在登录时，通过jwtService.sign()方法生成令牌
   ```bash
   const payload = { username: user.username, sub: user.id };
   return {
    access_token: this.jwtService.sign(payload),
   }
   ```
- 原理：
  - 用户登录时，服务器通过jwtService.sign()生成一个包含用户信息（如userId和username）的jwt令牌
  - 在后续请求中，用户需要在请求头中附带该令牌（如Authorization: Bearer <token>），服务器通过jwt策略对令牌进行验证，确保请求来自合法用户
- 拓展
  - 单点登录sso,一般对于单点登录的体系设计方案由三种：
    - cookie，常规系统可使用，不具备跨平台能力，无法通用化，例如在app中就无法使用
    - 单token（access_token），具备跨平台能力，但是缺乏安全性，因为token的失效不好控制
    - 双token（access_token和refresh_token），具备跨平台能力，安全性高，但是实现复杂度较高，需要考虑token的刷新，token的失效，token的续期等问题
    - token +cookie

# 说说如何使用@nestjs/schedule实现定时任务，并解释任务队列管理的原理
在nestjs中可以通过@nestjs/schedule和@nestjs/bull实现定时任务和任务队列的管理
- 定时任务：
   - 安装@nestjs/schedule包：
   ```bash
   npm install --save @nestjs/schedule
   ```
   - 在服务中引入cron装饰器来创建定时任务：
   ```bash
   @Injectable()
   export class CronExample {
     @Cron('45 * * * * *')
     handleCron() {
       console.log('Called when the second is 45');
     }
   }
   ```
- 任务队列管理：
  - 安装@nestjs/bull包:
   ```bash
   npm install --save @nestjs/bull bull
   ```
  - 配置任务队列:
   在app.module.ts中配置bull模块
   ```bash
   BullModule.forRoot({
    redis:{
     host: 'localhost',
     port: 6379,
    }
   })
   ```
  - 创建任务队列:
   ```bash
   @Injectable()
   export class QueueService {
     constructor(@InjectQueue('queueName') private readonly queue: Queue) {}

     async addTask(data: any) {
       const queue = this.queue.add('taskName', data);
    }
   }) 
   ```
  - 原理：  
   - 定时任务: 通过cron表达式定义任务的执行时间，当达到设定的时间时，任务自动触发执行
   - 任务队列：通过bull模块，任务可以被加入redis中的任务队列，并根据优先级或延时执行。队列支持任务的重试、失败处理和并发处理，适用于复杂的任务调度和分布式任务处理

# 镜像和容器的区别是什么？Dockerfile中cmd和run的区别?
镜像是只读的文件系统模版，包含运行应用所需的代码、运行时、系统工具、库和配置，类比面向对象编程中的“类”。容器是镜像的运行时实例，类比“对象”-- 一个对象可以同时启动多个互相隔离的容器，每个容器有自己的可写层和独立的进程空间。删除容器不会影响镜像，就像删除对象不会影响类定义一样.
run和cmd的核心区别在于执行时机：
- run 在docker build构建镜像时执行，每一条run指令会创建一个新的镜像层，通常用于安装软件包、编译代码、生成文件等构建操作。例如 run npm ci 会在构建时安装依赖并固化到镜像中。
- cmd 在docker run 启动容器时执行，是容器的默认启动命令，一个dockerfile中只有最后一条cmd生效.如果docker run 时手动指定了命令，会覆盖cmd，例如cmd["node","server.js"]定义了容器启动默认运行的进程
```yml
#RUN: 构建时执行，安装依赖并固化到镜像
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production

#CMD：启动时执行，容器的默认入口
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]

```
# Docker镜像为什么能做到小体积和层共享？分层文件系统的原理是什么？
Docker镜像采用联合文件系统（Union FS）实现分层结构。镜像不是一个单一的大文件，而是由多个只读层（layer）叠加而成，每一层记录相对于上一层的文件系统变更（新增、修改、删除）。多个镜像如果基于相同的基础层（比如都用node:20-alpine），这些基础层在磁盘上只存储一份，拉取时也只下载缺失的层，这就是“层共享”.
容器启动时，docker在所有只读层之上添加一层可写层（container layer），容器运行时的所有文件修改都写入这一层，底层只读层保持不变。这采用了写时复制（copy-on-write）机制:修改文件时先把文件从只读层复制到可写层再修改，删除文件时再可写层标记为“已删除”。删除容器时可写层消失，镜像层不受影响.
多阶段构建正是利用了分层特性：第一阶段用完整镜像安装编译工具、构建产物，第二阶段用轻量基础镜像只复制最终产物，构建工具和中间文件不会进入最终镜像，体积可以从1GB降到150MB
```yml
# 构建阶段：用完整镜像，包含编译工具
FROM node:20 as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 产物阶段：用轻量基础镜像，只复制最终产物
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/server.js"]
```
# Docker compose 解决了什么问题？depends_on是否真正能等待数据库就绪?D
Docker Compose解决了多容器应用的编排那提。一个真实应用通常包含web服务、数据库、缓存、消息队列等多个容器，手动用docker run 逐个启动需要处理端口映射、网络互通、启动顺序、环境变量等大量参数，停止和清理也容易遗漏。compose用一个docker-compose.yml文件声明所有服务及其依赖关系，一条docker compose up -d即可按顺序启动全部服务，一条docker compose down即可停止和清理全部服务.
depends_on只能控制容器启动顺序，不能等待服务“就绪”。它只保证postgresSql容器先于web容器启动，但postgresSql容器启动后用可能还需要几秒完成初始化（创建数据库、加载配置），此时web服务连接数据库会失败.这是compose最常见的坑之一。
正确做法是配合healthcheck使用：为数据库服务定义健康检查命令，然后再web服务的depends_on中指定conditin: service_healthy，这样compose会等待数据库服务健康检查通过后才启动web服务.同时应用代码中也应该加重试逻辑，双重保障
```yml  
services：
  web:
    build: .
    ports:
      - "3000:3000"
    environment: 
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/dbname
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

```
# k8s和Docker的关系是什么？？k3s为什么比标准k8s轻量？
Docker和kubernetes(k8s)不是竞争关系，而是互补关系。Docker是容器运行时（container runtime）,负责在单机上打包、运行和管理单个容器；k8s是容器编排系统（container orchestration），负责在多台机器组成的集群上调用和管理成百上千个容器。k8s本身不直接运行容器，它调用底层的容器运行时（早期用docker，现在默认用containerd）来实际启动和停止容器。可以理解为:Docker是“单个工人”，k8s是“工头”，工头指挥工人干活,
k3s是rancher（现suse）推出的轻量级k8s发行版，通过以下设计实现轻量化：
- 单二进制部署:k3s把k8s的所有组件打包成一个不到100MB的二进制文件，一条命令即可安装，而标准k8s需要分别安装kube-apiserver、etcd、kubelet等多个组件
- 用sqlite替代etcd:单节点场景下用嵌入式sqlite存储集群状态，省去了ectd的部署和运维开销，内存占用从2GB降至512MB以下
- 内置常用组件:内置了contiainerd、Traefik Ingress、CoreDNS、Local Path provisioner等常用组件，无需手动安装
k3s适合边缘计算、IoT设备、CI/CD流水线、本地开发和小型团队；大规模生产环境仍建议使用标准k8s或云商托管的EKS/GKE/AKS

