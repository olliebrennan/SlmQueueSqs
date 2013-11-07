SlmQueueSqs
===========

[![Build Status](https://travis-ci.org/juriansluiman/SlmQueueSqs.png?branch=master)](https://travis-ci.org/juriansluiman/SlmQueueSqs)
[![Latest Stable Version](https://poser.pugx.org/juriansluiman/slm-queue-sqs/v/stable.png)](https://packagist.org/packages/juriansluiman/slm-queue-sqs)
[![Latest Unstable Version](https://poser.pugx.org/juriansluiman/slm-queue-sqs/v/unstable.png)](https://packagist.org/packages/juriansluiman/slm-queue-sqs)

Created by Jurian Sluiman and Michaël Gallego

> NOTE : this is an early release of SlmQueueSqs, although it is tested, it may not work as expected. Please use
> with caution, and don't hesitate to open issues or PRs !


Requirements
------------
* [Zend Framework 2](https://github.com/zendframework/zf2)
* [SlmQueue](https://github.com/juriansluiman/SlmQueue)
* [Amazon AWS SDK > 2.1.1](https://github.com/aws/aws-sdk-php)
* [Amazon AWS ZF2 module](https://github.com/aws/aws-sdk-php-zf2)

To-do
-----

Feel free to help in those areas if you like this module !

* Write more tests to assert the queue work as expected
* Better error handling (currently, errors that may be returned by SQS client are completely ignored, we'd
 like to throw exceptions so that people can handle them in their code)
* More support for programmatic queue handling: currently SlmQueueSqs offers very few options to create new
 queues (we assume people to create them from the Amazon Console or directly through the SDK). It may be useful
 to offer better integration so that SlmQueueSqs also offers a nice interface to create new queues.

Installation
------------

First, install SlmQueue ([instructions here](https://github.com/juriansluiman/SlmQueue/blob/master/README.md)). Then,
add the following line into your `composer.json` file:

```json
"require": {
	"slm/queue-sqs": ">=0.2"
}
```

Then, enable the module by adding `SlmQueueSqs` in your application.config.php file (you must also add the `Aws` key
for enabling the AWS ZF2 module.

> Starting from 0.3.0, SlmQueueSqs now internally uses the official AWS Zend Framework 2 module, so you can write
your credentials only once for all AWS services.

Documentation
-------------

Before reading SlmQueueSqs documentation, please read [SlmQueue documentation](https://github.com/juriansluiman/SlmQueue).

Currently, SlmQueueSqs does not offer any real features to create queues. You'd better use the administrator console
of Amazon AWS services to create queues.

### Setting your AWS credentials

Please refer to [the documentation of the official AWS ZF2 module](https://github.com/aws/aws-sdk-php-zf2#configuration).

### Adding queues

SlmQueueSqs provides an interface for SQS queues that extends `SlmQueue\Queue\QueueInterface`, and provides in
addition the following methods:

* batchPush(array $jobs, array $options = array()): insert many jobs at once into the queue. Please note that if
you need to specify options, the index key for both jobs and options must matched.
* batchDelete(array $jobs): delete multiple jobs at once from the queue.

A concrete class that implements this interface is included: `SlmQueueSqs\Queue\SqsQueue` and a factory is available to
create Sqs queues. Therefore, if you want to have a queue called "email", just add the following line in your
`module.config.php` file:

```php
return array(
    'slm_queue' => array(
        'queues' => array(
            'factories' => array(
                'email' => 'SlmQueueSqs\Factory\SqsQueueFactory'
            )
        )
    )
);
```

This queue can therefore be pulled from the QueuePluginManager class.


### Operations on queues

> The name of the options match the names of the Amazon AWS SDK.

#### push / batchPush

Valid option is:

* delay_seconds: the duration (in seconds) the message has to be delayed

Example:

```php
$queue->push($job, array(
    'delay_seconds' => 20
));
```

#### pop

Valid options are:

* max_number_of_messages: maximum number of jobs to return. As of today, the max value can be 10. Please
 remember that Amazon SQS does not guarantee that you will receive exactly
 this number of messages, rather you can receive UP-TO n messages.
* visibility_timeout: the duration (in seconds) that the received messages are hidden from subsequent
  retrieve requests after being retrieved by a pop request
* wait_time_seconds: by default, when we ask for a job, it will do a "short polling", it will
  immediately return if no job was found. Amazon SQS also supports "long polling". This
  value can be between 1 and 20 seconds. This allows to maintain the connection active
  during this period of time, hence reducing the number of empty responses.

### Executing jobs

SlmQueueSqs provides a command-line tool that can be used to pop and execute jobs. You can type the following
command within the public folder of your Zend Framework 2 application:

`php index.php queue sqs <queueName> [--maxJobs=] [--visibilityTimeout=] [--waitTime=] --start`

The queueName is a mandatory parameter, while the other parameters are all optional:

* maxJobs: maximum number of jobs that can be returned from a single pop call
* visibilityTimeout: duration (in seconds) that the received messages are hidden from subsequent retrieve requests after being retrieved by a pop request
* waitTime: wait time (in seconds) for which the call will wait for a job to arrive in the queue before returning


### Getting the list of all existing queues

You may want to retrieve a list of all existing Amazon SQS queues. You can do so by creating the `SqsService` object
and calling the method `getQueueUrls`:

```php
$sqsService = $serviceLocator->get('SlmQueueSqs\Service\SqsService');
$queues     = $sqsService->getQueueUrls();
```
