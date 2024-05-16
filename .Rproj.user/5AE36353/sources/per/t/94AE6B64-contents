KIP-1041: Drop `offsets.commit.required.acks` config in 4.0 (deprecate
in 3.8)
================

- Created by [David
  Jacot](https://cwiki.apache.org/confluence/display/~dajac), last
  modified on [May 13,
  2024](https://cwiki.apache.org/confluence/pages/diffpagesbyversion.action?pageId=303794933&selectedPageVersions=2&selectedPageVersions=3 "Show changes")

- [Motivation](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=303794933#KIP1041:Drop%60offsets.commit.required.acks%60configin4.0(deprecatein3.8)-Motivation)

- [Public
  Interfaces](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=303794933#KIP1041:Drop%60offsets.commit.required.acks%60configin4.0(deprecatein3.8)-PublicInterfaces)

- [Proposed
  Changes](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=303794933#KIP1041:Drop%60offsets.commit.required.acks%60configin4.0(deprecatein3.8)-ProposedChanges)

- [Compatibility, Deprecation, and Migration
  Plan](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=303794933#KIP1041:Drop%60offsets.commit.required.acks%60configin4.0(deprecatein3.8)-Compatibility,Deprecation,andMigrationPlan)

- [Rejected
  Alternatives](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=303794933#KIP1041:Drop%60offsets.commit.required.acks%60configin4.0(deprecatein3.8)-RejectedAlternatives)

# Status

**Current state**: *Accepted*

**Discussion thread**:
[*here*](http://mail-archives.apache.org/mod_mbox/kafka-dev/201501.mbox/%3CCAOeJiJh6Vkkca85bWYgkeOZ8rC6%2BKDh7zzq8vMKECL_7PNExTA%40mail.gmail.com%3E)*  
*

**JIRA**: [*here*](https://issues.apache.org/jira/browse/KAFKA-16658)*  
*

Please keep the discussion on the mailing list rather than commenting on
the wiki (wiki discussions get unwieldy fast).

# Motivation

*While working on the new group coordinator backing KIP-848, we found
out about the `offsets.commit.required.acks` config. Its documentation
says that its default value (-1) should never be overridden. It is also
worth mentioning that whenever set to something different than -1, the
current implementation does not respect the semantic of high watermark,
like a regular consumer does. So for instance, if 1 is used, the
coordinator writes to the leader and then directly materialize the
offset to the cache. We suppose that it was implemented this way to
ensure that the last received offsets are directly available for next
offset fetch requests. Note that this is also wrongly used when writing
group metadata (rebalance) to the log. This guarantee provided by the
non default configurations are weak and inconsistent. As there are no
known use cases where it should be, we propose to simply remove it.*

# Public Interfaces

None.

# Proposed Changes

- Deprecate *`offsets.commit.required.acks`* in 3.8. As usually, the
  broker will log a warning if the configuration is used.

- Remove *`offsets.commit.required.acks`* in 4.0.

# Compatibility, Deprecation, and Migration Plan

- *If anyone uses this configuration, it would transparently moved to
  using the default (-1). As a side effect, we suppose that it could
  have a slightly impact on performance.*

# Rejected Alternatives

- *The alternative would be to keep it.*
