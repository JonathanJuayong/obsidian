One of the leading [[SIEM]] solutions. It allows users to collect, analyse, and correlate network and machine logs in real time.

# Components:

## 1. Forwarder

Its main task is to collect the data and send it to the Splunk instance. The forwarder collects the data from the log sources and sends it to the Indexer.

## 2. Indexer

Indexer plays the main role in processing the data it receives from forwarders. It parses and normalises the data into field-value pairs, categorises it, and stores the results as events, making the processed data easy to search and analyse.
## 3. Search Head

The place within the **Search & Reporting App** where users can search the indexed logs. The searches are done using the [[SPL|SPL or Search Processing Language]], a powerful query language for searching indexed data.