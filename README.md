# QcloudCmq
a tencent cloud queue library 

## 安装

使用 Composer 安装:

```
composer require "songyang/qcloud-cmq"
```

```
### 使用方式

use songyang\qcloud-cmq;


class QueueDemo
{
    private $secretId;
    private $secretKey;
    private $endpoint;
    public function __construct($secretId, $secretKey, $endpoint)
    {
        $this->secretId = $secretId;
        $this->secretKey = $secretKey;
        $this->endpoint = $endpoint;
    }
    public function run()
    {        
            $queue_name = "MySampleQueue1";
            $my_account = new Account($this->endpoint, $this->secretId, $this->secretKey);
            $my_queue = $my_account->get_queue($queue_name);
            $queue_meta = new QueueMeta();
            $queue_meta->queueName = $queue_name;
            $queue_meta->pollingWaitSeconds = 10;
            $queue_meta->visibilityTimeout = 10;
            $queue_meta->maxMsgSize = 1024;
            $queue_meta->msgRetentionSeconds = 3600;
            try
            { 
                $my_queue->create($queue_meta);
                echo "Create Queue Succeed! \n" . $queue_meta . "\n";
                $my_queue->set_attributes($queue_meta);
                echo "Set Queue Attributes Succeed! QueueMeta:" . $queue_meta . "\n";
                $result = $my_account->list_queue();
                echo "List Queue Succeed! result: " . json_encode($result) . "\n";
                $queue_meta = $my_queue->get_attributes();
                echo "Get Queue Attributes Succeed! QueueMeta:" . $queue_meta . "\n";
                $msg_body = "I am test message.";
                $msg = new Message($msg_body);
                $re_msg = $my_queue->send_message($msg);
                echo "Send Message Succeed! MessageBody:" . $msg_body . " MessageID:" . $re_msg->msgId . "\n";
                $recv_msg = $my_queue->receive_message(3);
                echo "Receive Message Succeed! " . $recv_msg . "\n";
                $messages = array();
                for ($i=0; $i<3; $i++) {
                    $msg_body = "I am test message " . $i;
                    $msg = new Message($msg_body);
                    $messages [] = $msg;
                                                    
                }
                $re_msg_list = $my_queue->batch_send_message($messages);
                echo "Batch Send Message Succeed! " . json_encode($re_msg_list);
                $wait_seconds = 3;
                $num_of_msg = 3;
                $recv_msg_list = $my_queue->batch_receive_message($num_of_msg, $wait_seconds);
                echo "Batch Receive Message Succeed! " . json_encode($recv_msg_list) . "\n";
                $my_queue->delete();
                                                                            
                }
                catch (CMQExceptionBase $e)
                {
                echo "Create Queue Fail! Exception: " . $e;
                return;
                                                                            
                }
    }
}


class TopicDemo
{
    private $secretId;
    private $secretKey;
    private $endpoint;
    public function __construct($secretId, $secretKey, $endpoint)
    {
        $this->secretId = $secretId;
        $this->secretKey = $secretKey;
        $this->endpoint = $endpoint;
    }
    public function run()
    {        
        try 
        {
            // create account and topic
            $topic_name = "Topic-test-php";
            echo "init account \n";
            $my_account = new Account($this->endpoint, $this->secretId, $this->secretKey);
            $my_topic = $my_account->get_topic($topic_name); 	
            $my_topicmeta = new TopicMeta();
            $my_topic->create($my_topicmeta);
            echo "get and set topic meta \n";
            $my_topicmeta = $my_topic->get_attributes();
            $my_topicmeta->maxMsgSize=1024;
            $my_topic->set_attributes($my_topicmeta);
            echo "set attributes\n";	
            // list topic
            $topiclist = $my_account->list_topic();
            echo $topiclist;
            // publish message and batch publish message without tags 
            $msg = "this is a test message ";
            $msgid = $my_topic->publish_message($msg);
            echo "publish message without tag \n";     
            $vmsg = array();
            for ($i=0; $i<3; $i++) {
                $msg ="I am test message ";
                $vmsg [] = $msg;
            }
            $vmsgid = $my_topic->batch_publish_message($vmsg);
            echo "batch publish message without tags \n" ;
            // publish message  with tags 
            // tag define 
            $vtag = array("test","cmq","york");
            $msg = "this is a test message";
            $msgid = $my_topic->publish_message($msg, $vtag);
            echo " publish message with tag \n" ;
            $vmsg = array();
            for ($i=0; $i<3; $i++) {
                $msg ="I am test message " . $i;
                $vmsg [] = $msg;
            }
            $vmsgid = $my_topic->batch_publish_message($vmsg, $vtag);
            echo "batch publish message with tag \n";
            // create subscription 
            $subscription_name = "subsc-test34324";
            $my_subscription = $my_account->get_subscription($topic_name, $subscription_name);
            $subscriptionmeta = new SubscriptionMeta();
            // get and set subscription meta 
			// please input your endpoint and protocol
            $subscriptionmeta->Endpoint="";
            $subscriptionmeta->Protocol="";
            $my_subscription->create($subscriptionmeta);
            echo "create sub \n";
            $subscriptionmeta = $my_subscription->get_attributes();
            echo "get attributes\n";
            echo $subscriptionmeta ; 
            $my_subscription->set_attributes($subscriptionmeta);
            echo "set attributes\n";            
            // list subscription 
            $subscriptionlist = $my_topic->list_subscription($topic_name);
            echo $subscriptionlist ;
            echo "list subscription \n"; 
            // delete subscription and topic 
            $my_subscription->delete();
            echo "delete subscription \n";
            $my_topic->delete();
            echo "delete topic \n";
        }
        catch(CMQExceptionBase $e)
        {
            echo $e;
        }
    }
}

```
