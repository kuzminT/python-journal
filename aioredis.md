# Redis pub/sub

## Articles

- [Basic Redis Usage Example - Part 1: Exploring PUB-SUB with Redis & Python](https://kb.objectrocket.com/redis/basic-redis-usage-example-part-1-exploring-pub-sub-with-redis-python-583)
- [Event Data Pipelines with Redis Pub/Sub, Async Python and Dash](https://itnext.io/event-data-pipelines-with-redis-pub-sub-async-python-and-dash-ab0a7bac63b0)


## Examples

### 1
	async def consumer(channel):
		    while await channel.wait_message():
		        msg = await channel.get(encoding='utf-8')
		        for connection in connections:
		            await connection.write_message(msg)


	async def setup():
	    connection = await aioredis.create_redis('redis://localhost')
	    channel = await connection.subscribe('notifications')
	    asyncio.ensure_future(consumer(channel))


	import asyncio
	import aioredis


	async def reader(ch):
	    while (await ch.wait_message()):
	        msg = await ch.get_json()
	        print("Got Message:", msg)

	async def main():
	    pub = await aioredis.create_redis(
	        'redis://localhost')
	    sub = await aioredis.create_redis(
	        'redis://localhost')
	    res = await sub.subscribe('chan:1')
	    ch1 = res[0]

	    tsk = asyncio.ensure_future(reader(ch1))

	    res = await pub.publish_json('chan:1', ["Hello", "world"])
	    assert res == 1

	    await sub.unsubscribe('chan:1')
	    await tsk
	    sub.close()
	    pub.close()


	if __name__ == '__main__':
    asyncio.run(main())


### 2

	import aioredis
	import asyncio


	async def main():
	    redis = await aioredis.create_redis_pool('redis://localhost')

	ch1, ch2 = await redis.subscribe('channel:1', 'channel:2')
	assert isinstance(ch1, aioredis.Channel)
	assert isinstance(ch2, aioredis.Channel)

	async def reader(channel):
	    async for message in channel.iter():
	        print("Got message:", message)
	asyncio.get_running_loop().create_task(reader(ch1))
	asyncio.get_running_loop().create_task(reader(ch2))

	await redis.publish('channel:1', 'Hello')
	await redis.publish('channel:2', 'World')

	redis.close()
	await redis.wait_closed()

	asyncio.run(main())


