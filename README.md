KIP-1036: Extend RecordDeserializationException exception
================

Created by [Damien
Gasparina](https://cwiki.apache.org/confluence/display/~d.gasparina),
last modified on [May 03,
2024](https://cwiki.apache.org/confluence/pages/diffpagesbyversion.action?pageId=301795741&selectedPageVersions=6&selectedPageVersions=7 "Show changes")

- [Status](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1036%3A+Extend+RecordDeserializationException+exception#KIP1036:ExtendRecordDeserializationExceptionexception-Status)

- [Motivation](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1036%3A+Extend+RecordDeserializationException+exception#KIP1036:ExtendRecordDeserializationExceptionexception-Motivation)

- [Public
  Interfaces](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1036%3A+Extend+RecordDeserializationException+exception#KIP1036:ExtendRecordDeserializationExceptionexception-PublicInterfaces)

- [Proposed
  Changes](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1036%3A+Extend+RecordDeserializationException+exception#KIP1036:ExtendRecordDeserializationExceptionexception-ProposedChanges)

  - [Usage
    example](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1036%3A+Extend+RecordDeserializationException+exception#KIP1036:ExtendRecordDeserializationExceptionexception-Usageexample)

- [Compatibility, Deprecation, and Migration
  Plan](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1036%3A+Extend+RecordDeserializationException+exception#KIP1036:ExtendRecordDeserializationExceptionexception-Compatibility,Deprecation,andMigrationPlan)

- [Test
  Plan](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1036%3A+Extend+RecordDeserializationException+exception#KIP1036:ExtendRecordDeserializationExceptionexception-TestPlan)

- [Rejected
  Alternatives](https://cwiki.apache.org/confluence/display/KAFKA/KIP-1036%3A+Extend+RecordDeserializationException+exception#KIP1036:ExtendRecordDeserializationExceptionexception-RejectedAlternatives)

# Status

**Current state**: *Under Discussion*

**Discussion thread**:
[*here*](https://lists.apache.org/thread/or85okygtfywvnsfd37kwykkq5jq7fy5)*  
*

**JIRA**: [*here*](https://issues.apache.org/jira/browse/KAFKA-16507)*  
*

Please keep the discussion on the mailing list rather than commenting on
the wiki (wiki discussions get unwieldy fast).

# Motivation

[KIP-334](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=87297793)
introduced into the Consumer the RecordDeserializationException with
offsets information. That is useful to skip a poison pill but as you do
not have access to the Record, it still prevents easy implementation of
dead letter queue or simply logging the faulty data.

# Public Interfaces

The changes in the RecordDeserializationException are a new constructor
with added argument timestampType, timestamp, keyBuffer, valueBuffer,
Headers and a new enum DeserializationExceptionOrigin.  New accessor
methods for the added fields.

A new enum DeserializationExceptionOrigin field is allowing to
differentiate if needed the origin of the error: KEY or VALUE.

As it’s only addition, it would still be compatible with existing
consumer code.

**RecordDeserializationException.java**

<table>
<colgroup>
<col style="width: 101%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>public</code> <code>class</code>
<code>RecordDeserializationException extends</code>
<code>SerializationException {</code></p>
<p><code>// New enum</code></p>
<p><code>public</code> <code>enum</code>
<code>DeserializationExceptionOrigin {</code></p>
<p><code>KEY,</code></p>
<p><code>VALUE</code></p>
<p><code>}</code></p>
<p><code>@Deprecated</code></p>
<p><code>public</code>
<code>RecordDeserializationException(TopicPartition partition,</code></p>
<p><code>long</code> <code>offset,</code></p>
<p><code>String message,</code></p>
<p><code>Throwable cause);</code></p>
<p><code>// New constructor</code></p>
<p><code>public</code>
<code>RecordDeserializationException(DeserializationExceptionOrigin origin,</code></p>
<p><code>TopicPartition partition,</code></p>
<p><code>long</code> <code>offset,</code></p>
<p><code>long</code> <code>timestamp,</code></p>
<p><code>TimestampType timestampType,</code></p>
<p><code>ByteBuffer keyBuffer,</code></p>
<p><code>ByteBuffer valueBuffer,</code></p>
<p><code>Headers headers,</code></p>
<p><code>String message,</code></p>
<p><code>Throwable cause);</code></p>
<p><code>// New methods</code></p>
<p><code>public</code>
<code>DeserializationExceptionOrigin origin();</code></p>
<p><code>public</code> <code>TimestampType timestampType();</code></p>
<p><code>public</code> <code>long</code> <code>timestamp();</code></p>
<p><code>public</code> <code>ByteBuffer keyBuffer();</code></p>
<p><code>public</code> <code>ByteBuffer valueBuffer();</code></p>
<p><code>public</code> <code>Headers headers();</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

# Proposed Changes

We propose to include record content and metadata to the
RecordDeserializationException. The keyBuffer() and valueBuffer()
methods will allow to access the content of the record. This would still
permit to access offsets and then skip the record but also to fetch the
concerned data for specific processing by sending it to a dead letter
queue or whatever action that makes sense.

### Usage example

Here is an example of basic usage to implement a DLQ feature

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>{</code></p>
<p><code>// …</code></p>
<p><code>try</code>
<code>(KafkaConsumer&lt;String, SimpleValue&gt; consumer = new</code>
<code>KafkaConsumer&lt;&gt;(settings())) {</code></p>
<p><code>// Subscribe to our topic</code></p>
<p><code>LOGGER.info("Subscribing to topic "</code>
<code>+ KAFKA_TOPIC);</code></p>
<p><code>consumer.subscribe(List.of(KAFKA_TOPIC));</code></p>
<p><code>LOGGER.info("Subscribed !");</code></p>
<p><code>try</code>
<code>(KafkaProducer&lt;byte[], byte[]&gt; dlqProducer = new</code>
<code>KafkaProducer&lt;&gt;(producerSettings())) {</code></p>
<p><code>//noinspection InfiniteLoopStatement</code></p>
<p><code>while</code> <code>(true) {</code></p>
<p><code>try</code> <code>{</code></p>
<p><code>final</code>
<code>var records = consumer.poll(POLL_TIMEOUT);</code></p>
<p><code>LOGGER.info("poll() returned {} records", records.count());</code></p>
<p><code>for</code> <code>(var record : records) {</code></p>
<p><code>LOGGER.info("Fetch record key={} value={}", record.key(), record.value());</code></p>
<p><code>// Any processing</code></p>
<p><code>// ...</code></p>
<p><code>}</code></p>
<p><code>} catch</code>
<code>(RecordDeserializationException re) {</code></p>
<p><code>long</code> <code>offset = re.offset();</code></p>
<p><code>Throwable t = re.getCause();</code></p>
<p><code>LOGGER.error("Failed to consume at partition={} offset={}", re.topicPartition().partition(), offset, t);</code></p>
<p><code>sendDlqRecord(dlqProducer, re);</code></p>
<p><code>LOGGER.info("Skipping offset={}", offset);</code></p>
<p><code>consumer.seek(re.topicPartition(), offset + 1);</code></p>
<p><code>} catch</code> <code>(Exception e) {</code></p>
<p><code>LOGGER.error("Failed to consume", e);</code></p>
<p><code>}</code></p>
<p><code>}</code></p>
<p><code>}</code></p>
<p><code>} finally</code> <code>{</code></p>
<p><code>LOGGER.info("Closing consumer");</code></p>
<p><code>}</code></p>
<p><code>}</code></p>
<p><code>void</code>
<code>sendDlqRecord(KafkaProducer&lt;byte[], byte[]&gt; dlqProducer, RecordDeserializationException re) {</code></p>
<p><code>var dlqRecord = new</code>
<code>ProducerRecord&lt;&gt;(DLQ_TOPIC, Utils.toNullableArray(re.keyBuffer()), Utils.toNullableArray(re.valueBuffer()));</code></p>
<p><code>try</code> <code>{</code></p>
<p><code>dlqProducer.send(dlqRecord).get();</code></p>
<p><code>LOGGER.info("Record sent to DLQ");</code></p>
<p><code>} catch</code> <code>(Exception e) {</code></p>
<p><code>LOGGER.error("Failed to send corrupted record to DLQ", e);</code></p>
<p><code>}</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

# Compatibility, Deprecation, and Migration Plan

This change is backward compatible and will have no impact on existing
application.

The previous RecordDeserializationException constructor is deprecated as
no more used.

# Test Plan

- Unit test CompletedFetchTest

# Rejected Alternatives

- Using a raw byte array consumer and doing the deserialization in the
  user code. This permits dead letter queue implementation but is moving
  all the complexity of deserialization to user’s code.
