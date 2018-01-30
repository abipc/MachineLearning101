
.. _h635651331301530583812d37d654d:

Kuyil API Document
##################

Kuyil is a data management library. It provides an API interface to the clients (processes) to access, update and manage data. Kuyil API interface allows concurrent process to access and update data (read and write) in an efficient way without any data corruption. It also provides logging information around the use of datasets which can be very useful for accounting and troubleshooting. 

.. _h166758687a4f5016506f6536958375:

Entities
********

.. _h334e4d14467a2733651c726db5e:

Repository
==========

	Id

	Name

	Config

.. _h25f73491356214a61732f6d2f655a:

Dataset 
========

    Id

    Class

    Instance

    Label

    Version

    Tags

    Alt_uri

    Read_lock_ts

    Write_lock_id

    Write_lock_ts

    Is_expired

    Is_valid

    Is_incomplete

    Is_marked_for_deletion

    Is_deleted

    Created_ts

    Modified_ts

    Dataset_created_ts

    Dataset_ready_ts

    Dataset_modified_ts

    Dataset_marked_for_deletion_ts

    Dataset_deleted_ts

    Dataset_expired_ts

    Dataset_restored_ts

    Comment

    Metadata

    Bytes

    Lines

.. _h25307437306095a7f34d1c712f2c4c:

Dataset_Repository
==================

This class represents many to many relationship between dataset and repository. One dataset can be present in more than one repository at a time (Use case - copy dataset from one repo to another). Also, each repository can have multiple datasets.

Id

Dataset_id

Repository_id

.. _h2ac7b173b4a4b7a6b32256c157f4d66:

Dataset_History
===============

.. _h3a7a354276d7b684e21623852417160:

Public Classes
**************

.. _h737f78c801d59376058703835e7c45:

Kuyil
=====

	This is the main entry point for Kuyil.

.. _h334e4d14467a2733651c726db5e:

Repository
==========

	This is the abstraction for repository object. The following repositories are supported as of now -

#. AWS S3

.. _h236a5820161d2b306a19192e80596114:

Dataset
=======

	This is the abstraction for dataset entity. A dataset is uniquely defined by class, instance label and version of the content. Version is a private member. Datasets are versioned which allows one process to make update to the dataset without affecting other processes which are still reading the older version of the dataset.

.. _h2ac7b173b4a4b7a6b32256c157f4d66:

Dataset_History
===============

	This is the trail of all actions - updates, creations and deletions done on Dataset object during its lifetime. This enables accounting and troubleshooting. 

.. _h405552c64828205c3a4cd47476a42:

API Details
***********

.. _h171fc1f72712d2e7a76746c696f417a:

Class Kuyil()
-------------

This is the top level object. Primary use is to load and access Repository and Dataset object.

.. _h1c203b3f7d1b334eb2ff603797d58:

kuyil.get_repository(name)
~~~~~~~~~~~~~~~~~~~~~~~~~~

	Returns a repository object with the name provided.

.. _hf39401797f7074676e6e7b6571050:

kuyil.get_datasets(class,instance_regex,label=None,repository,all_versions=False)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Returns a list of all Dataset objects that match the parameters. You can provide instance regex in standard regex form (find term for it - python regex format). If all_versions is set to False, only the latest version is returned.

.. _h3f3460106b4124d354e7623105bc12:

kuyil.create_dataset(class,instance,label,repository, comment=None, metadata=None, only_create=False)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Class, instance, label and version uniquely define a dataset. 

#. Version is not a public member. We use timestamp (epoch) for versions. 

#. For a new dataset, is_valid=False & is_incomplete=False. 

#. This API does not create a row in the database. A row is created when a write lock is acquired. 

#. Active version of the dataset is defined by the latest version of the dataset row in database for which Is_valid=True && Is_incomplete=False.

#. The use of only_create parameter is described below

    #. only_create=False

        #. If an active dataset row is already present, DO NOT create a new dataset rather return the Dataset object corresponding to the latest active version.

        #. If an active dataset row is not already present, create a new Dataset object. Do not create a row unless write lock is acquired.  

    #. only_create=True

        #. Always CREATE a new Dataset with a new version. Return this Dataset object. Do not create a row unless write lock is acquired.

.. _h5619301d28607d5757357558067212e:

kuyil.replace_dataset_in_label(class,instance,source_label, target_label, sink_label, keep_target_modified_ts=True)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Does two label changes on two datasets simultaneously ( source -> target AND target -> sink ). If keep_target_modified_ts is not specified (default = true), the source dataset copies to modified_ts of the dataset it replaces. There is no copy of data in this API rather only metadata changes. The latest active version for both source and target dataset is assumed. 

.. _h4f7c2024552227733027655b66716e65:

Class Repository()
------------------

	Repository object is an abstraction for the underlying data stores like AWS S3, MySQL, Google Cloud Storage.

	

.. _h207f5c0c7f2628f1e3e483f7c1346:

Repository().init
~~~~~~~~~~~~~~~~~

    This will initialize the underlying repository client (concrete implementation - example s3 ). All client configurations and access are part of the config which is a static configuration. 

.. _h4e36324733417f124d5b4b1a3d607f78:

Repository().create_uri(class, instance, id, overwrite=False, \*\*kwargs)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Returns URI/None based on success of the operation. Creates a physical URI (like s3 path) at the repository. 

    URI is made up of class, instance and id that uniquely defines a row in database.

.. _h297b805123601c65b577f484b171156:

Repository().get_resource_uri(class,instance,id)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    Returns URI string if it exists. Involves repository look up to resolve the path. None is returned if not present at the repository.

.. _h6420b3f271e3d3330f06956144277:

Repository().does_uri_exist(class, instance, id, \*\*kwargs)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Returns true or false. Resolve this by look up on repository. If a file is present at the URI and a corresponding entry exists in database, returns true otherwise returns false.

.. _h7448322330664b1351425c5d7b717767:

Repository().delete_uri(class, instance, id) 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Deletes the URI from repository and returns true or false on the basis of the success of the operation. This API must return false if there are any active locks on the Dataset which uses this URI.

.. _h6542744f78194b3d1e404523747d6a:

Repository().expire_uri(class, instance, id)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _h2c1d74277104e41780968148427e:




.. _h492a5d117137429384929646c314a6a:

Repository().restore_uri(class, instance, id) 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _hd4b21d613d533a496d75d4e5a3b:

Class Dataset()
---------------

\ |STYLE0|\ 

	Returns the new timestamp (time in epoch till which read lock is acquired) if the process is able to acquire a read lock. Updates read_lock_ts to current epoch + 5 mins. 

\ |STYLE1|\  

	Returns new timestamp (epoch till which write lock is acquired) on acquiring a write lock if there is no read or write. Write_lock_ts=current time epoch + 5 mins. Otherwise none.

.. _h2d666034226c37f667e78550551f4e:

Dataset().get_write_lock_if_read_or_idle(pid)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Returns new timestamp (epoch till which write lock is acquired) on acquiring a write lock if there is no write. Write_lock_ts=current time epoch + 5 mins. Otherwise none.

.. _h2c1d74277104e41780968148427e:




.. _h4f1c70533e4975f1954e2a746769e:

Dataset().get_write_lock_if_write_read_or_idle(pid)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

	Returns new timestamp (epoch till which write lock is acquired) on acquiring a write lock in any scenario (write/read/idle). Write_lock_ts=current time epoch + 5 mins. Otherwise none.

\ |STYLE2|\ 

	Increase the read_lock_ts by 5 mins and return this expiry timestamp. 

\ |STYLE3|\ 

    Increase write_lock_ts by 5 mins and return this expiry timestamp. A pid can maintain write lock if and only if this pid itself holds the write lock already. None otherwise.

\ |STYLE4|\ 

	Returns true if write_lock_ts > now.

\ |STYLE5|\ 

	Returns true if read_lock_ts > now.

\ |STYLE6|\ 

	Returns the URI for this dataset object. Internally uses repository.get_uri() 

\ |STYLE7|\ 

    Updates the is_valid flag to the given value.

\ |STYLE8|\ 

	Check if no version of this dataset is being read or write. If so, purge repository URI. Mark all versions deleted from mysql. This means it is inactive. Return None for example if get_dataset is called.

\ |STYLE9|\ 

	This API allows updating a comment if present. 

\ |STYLE10|\ 

	This API updates a comment if present, updates Dataset_write_ts to current time epoch, update Is_incomplete=False, write_lock_ts=current time epoch IFF the dataset was already write locked by the same pid.

\ |STYLE11|\ 

	This API updates a comment if present, updates Dataset_write_ts to None, update Is_incomplete=True, write_lock_ts=current time epoch IFF the dataset was already write locked by the same pid.

\ |STYLE12|\ 

    This API returns a dictionary of key values that make up the metadata for this dataset.

\ |STYLE13|\ 

	This API sets the metadata.

\ |STYLE14|\ 

	This API allows add/update any key value pair that makes up the metadata.

\ |STYLE15|\ 

	This API returns the class associated with this dataset.

\ |STYLE16|\ 

	This API returns the instance associated with this dataset.

\ |STYLE17|\ 

	This API returns the label associated with this dataset.

.. _h2c1d74277104e41780968148427e:




.. _h2c1d74277104e41780968148427e:





.. bottom of content


.. |STYLE0| replace:: **Dataset().get_read_lock()**

.. |STYLE1| replace:: **Dataset().get_write_lock_if_idle(pid)**

.. |STYLE2| replace:: **Dataset().maintain_read_lock()**

.. |STYLE3| replace:: **Dataset().maintain_write_lock(pid)**

.. |STYLE4| replace:: **Dataset().is_write_locked()**

.. |STYLE5| replace:: **Dataset().is_read_locked()**

.. |STYLE6| replace:: **Dataset().get_resource_uri()**

.. |STYLE7| replace:: **Dataset().set_validity(validity)**

.. |STYLE8| replace:: **Dataset().set_deleted(is_deleted)**

.. |STYLE9| replace:: **Dataset().read_finished(pid, comment=None)**

.. |STYLE10| replace:: **Dataset().write_success(pid, comment=None)**

.. |STYLE11| replace:: **Dataset().write_failed(pid, comment=None)**

.. |STYLE12| replace:: **Dataset().get_metadata()**

.. |STYLE13| replace:: **Dataset().set_metadata(dict)**

.. |STYLE14| replace:: **Dataset().set_metadata_key_value(key,value)**

.. |STYLE15| replace:: **Dataset().get_class()**

.. |STYLE16| replace:: **Dataset().get_instance()**

.. |STYLE17| replace:: **Dataset().get_label()**
