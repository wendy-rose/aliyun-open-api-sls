阿里云日志开放API
==============================
此包来源lokielse/aliyun-open-api-sls，在此基础上修复了bug

## Install

Via Composer

``` bash
$ composer require wendy/aliyun-open-api-sls
```

## Usage

```php
/**
 * Config
 */
$config = [
	'AccessKeyId'     => 'your_access_key_id',
	'AccessKeySecret' => 'your_access_key_secret',
	'endpoint'        => 'cn-hangzhou.sls.aliyuncs.com',
	'project'         => 'your_sls_project_name',
	'logStore'        => 'your_sls_log_store_name',
	'topic'           => '',
	'source'          => '',
];

/**
 * 写日志 (Write Logs)
 */
$logs = array(
	new LogItem([ 'user' => 'jello', 'action' => 'trash_photo', 'object_id' => 123456 ]),
	new LogItem([ 'user' => 'frank', 'action' => 'delete_user', 'object_id' => 100236 ]),
);

$putLogsRequest  = new PutLogsRequest($config['project'], $config['logStore'], $config['topic'], $config['source'], $logs);
$client          = new Client($config['endpoint'], $config['AccessKeyId'], $config['AccessKeySecret']);
$putLogsResponse = $client->putLogs($putLogsRequest);

/**
 * 读日志  (Read Logs)
 * 在控制台查看日志有3-5分钟延迟, 但是使用该SDK查看无延迟
 */
$listShardRequest = new ListShardsRequest($config['project'], $config['logStore']);

$listShardResponse = $client->listShards($listShardRequest);

foreach ($listShardResponse->getShardIds() as $shardId) {
	/**
	 * 对每一个ShardId，先获取Cursor
	 */
	$getCursorRequest = new GetCursorRequest($config['project'], $config['logStore'], $shardId, null, time() - 60);
	$response         = $client->getCursor($getCursorRequest);
	$cursor           = $response->getCursor();
	$count            = 100;

	while (true) {
		/**
		 * 从cursor开始读数据
		 */
		$batchGetDataRequest = new BatchGetLogsRequest($config['project'], $config['logStore'], $shardId, $count, $cursor);

		var_dump($batchGetDataRequest);

		$response = $client->batchGetLogs($batchGetDataRequest);

		if ($cursor == $response->getNextCursor()) {
			break;
		}

		$logGroupList = $response->getLogGroupList();

		/**
		 * @var \Aliyun\SLS\Log\LogGroup  $logGroup
		 * @var Aliyun\SLS\Log\Log        $log
		 * @var Aliyun\SLS\Log\LogContent $content
		 */
		foreach ($logGroupList as $logGroup) {

			print ( $logGroup->getCategory() );

			foreach ($logGroup->getLogsArray() as $log) {
				foreach ($log->getContentsArray() as $content) {
					print( $content->getKey() . ":" . $content->getValue() . "\t" );
				}
				print( "\n" );
			}
		}
		$cursor = $response->getNextCursor();
	}
}
```
[官方文档](https://help.aliyun.com/document_detail/29074.html)
