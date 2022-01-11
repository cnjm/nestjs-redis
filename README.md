# nestjs-redis 8 fix

# Redis

## Usage

 在app模块下导入:

```TypeScript
import {
    RedisModule
} from "@chenjm/nestjs-redis";

@Module({
    imports: [
        RedisModule.forRoot({
            closeClient: true,
            readyLog: true,
            config: {
                namespace: 'default',
                host: '127.0.0.1',
                port: 6380,
                password: 'redispassword'
            }
        })
    ]
})
export class AppModule {}
```

也可以配合 @nestjs/config 通过configService 提供配置

```TypeScript
import { ConfigModule, ConfigService } from "@nestjs/config";
import { RedisModule } from "@chenjm/nestjs-redis";

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      cache: true,
      envFilePath: `${process.env.NODE_ENV}.env`,
      load: [configuration],
    }),
    RedisModule.forRootAsync({
      inject: [ConfigService],
      useFactory: (configService: ConfigService) => {
        return {
          closeClient: true,
          readyLog: true,
          config: configService.get("redis"),
        };
      },
    }),
  ],
})
export class AppModule {}

```

提示：如果不为客户机设置名称空间，则其名称空间将设置为默认名称空间。请注意，您不应该有多个没有命名空间的客户端，或者具有相同的命名空间，否则它们将被覆盖。

## 我们可以通过两种方式使用redis. 也可以封装下自己常用的方法

 decorator 中使用:

```TypeScript
import { Injectable } from '@nestjs/common';
import { InjectRedis, DEFAULT_REDIS_NAMESPACE } from '@chenjm/nestjs-redis';
import { Redis } from 'ioredis';

@Injectable()
export class AppService {
    constructor(
        @InjectRedis() private readonly redis: Redis
        // or
        // @InjectRedis(DEFAULT_REDIS_NAMESPACE) private readonly redis: Redis
    ) {}

    async ping(): Promise<string> {
        return await this.redis.ping();
    }
}
```

 service中使用:

```TypeScript
import { Injectable } from '@nestjs/common';
import { RedisService, DEFAULT_REDIS_NAMESPACE } from '@chenjm/nestjs-redis';
import { Redis } from 'ioredis';

@Injectable()
export class AppService {
    private readonly redis: Redis;

    constructor(private readonly redisService: RedisService) {
        this.redis = this.redisService.getClient();
        // or
        // this.redis = this.redisService.getClient(DEFAULT_REDIS_NAMESPACE);
    }

    async ping(): Promise<string> {
        return await this.redis.ping();
    }
}
```
