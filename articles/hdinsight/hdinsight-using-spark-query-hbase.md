---
title: 'Verwenden von Spark zum Lesen und Schreiben von HBase-Daten: Azure HDInsight'
description: Verwenden Sie den Spark HBase-Connector, um Daten aus einem Spark-Cluster zu lesen und in einen HBase-Cluster zu schreiben.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 03/12/2019
ms.openlocfilehash: e3f5cb726dddbdbfbd1b1f48c800ac681e7a174c
ms.sourcegitcommit: 44a85a2ed288f484cc3cdf71d9b51bc0be64cc33
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/28/2019
ms.locfileid: "64696550"
---
# <a name="use-apache-spark-to-read-and-write-apache-hbase-data"></a>Verwenden von Apache Spark zum Lesen und Schreiben von Apache HBase-Daten

Apache HBase wird üblicherweise über die Low-Level-API (scan-, get- und put-Abfragen) oder mit einer SQL-Syntax unter Verwendung von Apache Phoenix abgefragt. Apache stellt auch den Apache Spark HBase-Connector bereit, der eine praktische und leistungsfähige Alternative zum Abfragen und Ändern von Daten darstellt, die von HBase gespeichert wurden.

## <a name="prerequisites"></a>Voraussetzungen

* Zwei separate installierte HDInsight-Cluster: ein HBase-Cluster und ein Spark-Cluster mit Spark 2.1 (HDInsight 3.6) als Mindestversion.
* Der Spark-Cluster muss mit minimaler Latenz direkt mit dem HBase-Cluster kommunizieren, daher besteht die empfohlene Konfiguration darin, beide Cluster im gleichen virtuellen Netzwerk bereitzustellen. Weitere Informationen finden Sie unter [Erstellen von Linux-basierten Clustern in HDInsight mithilfe des Azure-Portals](hdinsight-hadoop-create-linux-clusters-portal.md).
* Einen SSH-Client. Weitere Informationen finden Sie unter [Herstellen einer Verbindung mit HDInsight (Hadoop) per SSH](hdinsight-hadoop-linux-use-ssh-unix.md).
* Das [URI-Schema](hdinsight-hadoop-linux-information.md#URI-and-scheme) für Ihren primären Clusterspeicher. Dies ist „wasb://“ für Azure Blob Storage, „abfs://“ für Azure Data Lake Storage Gen2 oder „adl://“ für Azure Data Lake Storage Gen1. Wenn die sichere Übertragung für Blob Storage oder Data Lake Storage Gen2 aktiviert ist, lautet der URI „wasbs://“ bzw. „abfss://“. Siehe auch die Informationen zur [sicheren Übertragung](../storage/common/storage-require-secure-transfer.md).


## <a name="overall-process"></a>Übersicht über den Prozess

Der allgemeine Prozess zum Aktivieren Ihres Spark-Clusters für die Abfrage Ihres HDInsight-Clusters sieht wie folgt aus:

1. Bereiten Sie einige Beispieldaten in HBase vor.
2. Rufen Sie die Datei „hbase-site.xml“ aus dem Konfigurationsordner Ihres HBase-Clusters ab (/etc/hbase/conf).
3. Speichern Sie eine Kopie von „hbase-site.xml“ in Ihrem Spark 2-Konfigurationsordner (/etc/spark2/conf).
4. Führen Sie `spark-shell` aus, und verweisen Sie in der Option `packages` mit den Maven-Koordinaten auf den Spark HBase-Connector.
5. Definieren Sie einen Katalog, der das Spark-Schema zu HBase zuordnet.
6. Interagieren Sie entweder über die RDD- oder die Dataframe-APIs mit den HBase-Daten.

## <a name="prepare-sample-data-in-apache-hbase"></a>Vorbereiten von Beispieldaten in Apache HBase

In diesem Schritt erstellen Sie einfache Tabelle in Apache HBase und füllen sie auf. Diese Tabelle können Sie dann mit Spark abfragen.

1. Stellen Sie über SSH eine Verbindung mit dem Hauptknoten Ihres HBase-Clusters her. Weitere Informationen finden Sie unter [Herstellen einer Verbindung mit HDInsight per SSH](hdinsight-hadoop-linux-use-ssh-unix.md).  Bearbeiten Sie den unten angegebenen Befehl, indem Sie `HBASECLUSTER` durch den Namen Ihres HBase-Clusters und `sshuser` durch den Namen des SSH-Benutzerkontos ersetzen und anschließend den Befehl eingeben.

    ```
    ssh sshuser@HBASECLUSTER-ssh.azurehdinsight.net
    ```

2. Geben Sie den folgenden Befehl ein, um die HBase-Shell zu starten:

        hbase shell

3. Geben Sie den folgenden Befehl ein, um die Tabelle `Contacts` mit den Spaltenfamilien `Personal` und `Office` zu erstellen:

        create 'Contacts', 'Personal', 'Office'

4. Geben Sie die folgenden Befehle ein, um einige Beispieldatenzeilen zu laden:

        put 'Contacts', '1000', 'Personal:Name', 'John Dole'
        put 'Contacts', '1000', 'Personal:Phone', '1-425-000-0001'
        put 'Contacts', '1000', 'Office:Phone', '1-425-000-0002'
        put 'Contacts', '1000', 'Office:Address', '1111 San Gabriel Dr.'
        put 'Contacts', '8396', 'Personal:Name', 'Calvin Raji'
        put 'Contacts', '8396', 'Personal:Phone', '230-555-0191'
        put 'Contacts', '8396', 'Office:Phone', '230-555-0191'
        put 'Contacts', '8396', 'Office:Address', '5415 San Gabriel Dr.'

5. Geben Sie den folgenden Befehl ein, um die HBase-Shell zu beenden:

        exit 

## <a name="copy-hbase-sitexml-to-spark-cluster"></a>Kopieren von „hbase-site.xml“ in den Spark-Cluster
Kopieren Sie die Datei „hbase-site.xml“ aus dem lokalen Speicher in das Stammverzeichnis des Standardspeichers Ihres Spark-Clusters.  Bearbeiten Sie den Befehl, um ihn an Ihre Konfiguration anzupassen.  Geben Sie anschließend in Ihrer geöffneten SSH-Sitzung diesen Befehl für den HBase-Cluster ein:

| Syntaxwert | Neuer Wert|
|---|---|
|[URI-Schema](hdinsight-hadoop-linux-information.md#URI-and-scheme) | Nehmen Sie Änderungen zur Anpassung an Ihren Speicher vor.  Die unten angegebene Syntax gilt für Blobspeicher mit aktivierter sicherer Übertragung.|
|`SPARK_STORAGE_CONTAINER`|Fügen Sie den Namen Ihres Standardspeichercontainers ein, der für den Spark-Cluster verwendet wird.|
|`SPARK_STORAGE_ACCOUNT`|Fügen Sie den Namen Ihres Standardspeicherkontos ein, der für den Spark-Cluster verwendet wird.|

```
hdfs dfs -copyFromLocal /etc/hbase/conf/hbase-site.xml wasbs://SPARK_STORAGE_CONTAINER@SPARK_STORAGE_ACCOUNT.blob.core.windows.net/
```

## <a name="put-hbase-sitexml-on-your-spark-cluster"></a>Ablegen von „hbase-site.xml“ in Ihrem Spark-Cluster

1. Stellen Sie über SSH eine Verbindung mit dem Hauptknoten Ihres Spark-Clusters her.

2. Geben Sie den folgenden Befehl ein, um `hbase-site.xml` aus dem Standardspeicher Ihres Spark-Clusters in den Spark 2-Konfigurationsordner im lokalen Speicher des Clusters zu kopieren:

        sudo hdfs dfs -copyToLocal /hbase-site.xml /etc/spark2/conf

## <a name="run-spark-shell-referencing-the-spark-hbase-connector"></a>Ausführen der Spark-Shell und Verweisen auf den Spark HBase-Connector

1. Geben Sie in Ihrer geöffneten SSH-Sitzung für den Spark-Cluster den folgenden Befehl ein, um eine Spark-Shell zu starten:

    ```
    spark-shell --packages com.hortonworks:shc-core:1.1.1-2.1-s_2.11 --repositories https://repo.hortonworks.com/content/groups/public/
    ```  

2. Lassen Sie diese Instanz der Spark-Shell geöffnet, und fahren Sie mit dem nächsten Schritt fort.

## <a name="define-a-catalog-and-query"></a>Definieren von Katalog und Abfrage

In diesem Schritt definieren Sie ein Katalogobjekt, das das Apache Spark-Schema Apache HBase zuordnet.  

1. Geben Sie in Ihrer geöffneten Spark-Shell die folgenden `import`-Anweisungen ein:

    ```scala
    import org.apache.spark.sql.{SQLContext, _}
    import org.apache.spark.sql.execution.datasources.hbase._
    import org.apache.spark.{SparkConf, SparkContext}
    import spark.sqlContext.implicits._
    ```  

2. Geben Sie den folgenden Befehl ein, um einen Katalog für die Tabelle „Kontakte“ zu definieren, die Sie in HBase erstellt haben:

    ```scala
    def catalog = s"""{
        |"table":{"namespace":"default", "name":"Contacts"},
        |"rowkey":"key",
        |"columns":{
        |"rowkey":{"cf":"rowkey", "col":"key", "type":"string"},
        |"officeAddress":{"cf":"Office", "col":"Address", "type":"string"},
        |"officePhone":{"cf":"Office", "col":"Phone", "type":"string"},
        |"personalName":{"cf":"Personal", "col":"Name", "type":"string"},
        |"personalPhone":{"cf":"Personal", "col":"Phone", "type":"string"}
        |}
    |}""".stripMargin
    ```

    Mit dem Code wird Folgendes durchgeführt:  

     a. Definieren Sie ein Katalogschema für die HBase-Tabelle namens `Contacts`.  
     b. Identifizieren Sie das rowkey-Element als `key`, und ordnen Sie die in Spark verwendeten Spaltennamen der Spaltenfamilie, dem Spaltennamen und dem Spaltentyp zu, die in HBase verwendet werden.  
     c. Das rowkey-Element muss im Detail auch als benannte Spalte (`rowkey`) definiert werden, die über eine spezifische `cf`-Spaltenfamilie `rowkey` verfügt.  

3. Geben Sie den folgenden Befehl ein, um eine Methode zu definieren, die einen Datenrahmen um Ihre `Contacts`-Tabelle in HBase bereitstellt:

    ```scala
    def withCatalog(cat: String): DataFrame = {
        spark.sqlContext
        .read
        .options(Map(HBaseTableCatalog.tableCatalog->cat))
        .format("org.apache.spark.sql.execution.datasources.hbase")
        .load()
     }
    ```

4. Erstellen Sie eine Instanz des Datenrahmens:

    ```scala
    val df = withCatalog(catalog)
    ```  

5. Fragen Sie den Datenrahmen ab:

    ```scala
    df.show()
    ```

6. Es sollten zwei Zeilen mit Daten angezeigt werden:

        +------+--------------------+--------------+-------------+--------------+
        |rowkey|       officeAddress|   officePhone| personalName| personalPhone|
        +------+--------------------+--------------+-------------+--------------+
        |  1000|1111 San Gabriel Dr.|1-425-000-0002|    John Dole|1-425-000-0001|
        |  8396|5415 San Gabriel Dr.|  230-555-0191|  Calvin Raji|  230-555-0191|
        +------+--------------------+--------------+-------------+--------------+

7. Registrieren Sie eine temporäre Tabelle, sodass Sie die HBase-Tabelle mithilfe von Spark SQL abfragen können:

    ```scala
    df.createTempView("contacts")
    ```

8. Erstellen Sie eine SQL-Abfrage für die `contacts`-Tabelle:

    ```scala
    val query = spark.sqlContext.sql("select personalName, officeAddress from contacts")
    query.show()
    ```

9. Die Ergebnisse sollten wie folgt aussehen:

        +-------------+--------------------+
        | personalName|       officeAddress|
        +-------------+--------------------+
        |    John Dole|1111 San Gabriel Dr.|
        |  Calvin Raji|5415 San Gabriel Dr.|
        +-------------+--------------------+

## <a name="insert-new-data"></a>Einfügen neuer Daten

1. Um einen neuen Kontaktdatensatz einzufügen, definieren Sie eine `ContactRecord`-Klasse:

    ```scala
    case class ContactRecord(
        rowkey: String,
        officeAddress: String,
        officePhone: String,
        personalName: String,
        personalPhone: String
        )
    ```

2. Erstellen Sie eine Instanz von `ContactRecord`, und fügen Sie sie in ein Array ein:

    ```scala
    val newContact = ContactRecord("16891", "40 Ellis St.", "674-555-0110", "John Jackson","230-555-0194")

    var newData = new Array[ContactRecord](1)
    newData(0) = newContact
    ```

3. Speichern Sie das Array mit den neuen Daten in HBase:

    ```scala
    sc.parallelize(newData).toDF.write.options(Map(HBaseTableCatalog.tableCatalog -> catalog, HBaseTableCatalog.newTable -> "5")).format("org.apache.spark.sql.execution.datasources.hbase").save()
    ```

4. Untersuchen Sie die Ergebnisse:

    ```scala  
    df.show()
    ```

5. Es sollte in etwa folgende Ausgabe angezeigt werden:

        +------+--------------------+--------------+------------+--------------+
        |rowkey|       officeAddress|   officePhone|personalName| personalPhone|
        +------+--------------------+--------------+------------+--------------+
        |  1000|1111 San Gabriel Dr.|1-425-000-0002|   John Dole|1-425-000-0001|
        | 16891|        40 Ellis St.|  674-555-0110|John Jackson|  230-555-0194|
        |  8396|5415 San Gabriel Dr.|  230-555-0191| Calvin Raji|  230-555-0191|
        +------+--------------------+--------------+------------+--------------+

## <a name="next-steps"></a>Nächste Schritte

* [Apache Spark HBase-Connector](https://github.com/hortonworks-spark/shc)
