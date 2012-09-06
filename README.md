CouchSource
===========

CouchSource is a [CakePHP datasource](http://book.cakephp.org/view/1075/DataSources) allowing to build Models from a [CouchDB](http://couchdb.apache.org/) database.  
This is an update/fix for the https://github.com/PathMotion/cakephp-couchsource to work on CakePHP 2.1+

Install
-------

After having [set the couchdb](http://wiki.apache.org/couchdb/Installation) engine, follow this steps to make it work with CakePHP :

* Put the *couch_source.php* file into the *datasource* directory, usually <code>app/models/datasource</code>.
* Edit the *database.php* configuration file to add the following :

		class DATABASE_CONFIG {
		
			// [...]
		
			var $couch = array(
				'datasource' => 'couch',
				'host' => '127.0.0.1',
				'port' => 5984,
				'user' => 'my_couchdb_user',
				'password' => 'my_couchdb_password',
				'auth_method' => 'cookie' // (or 'basic')
			);
	
		}
	
* create a model that you want to use CouchDB

		class MyModel extends Model {

			public $name = 'MyModel';
			public $useDbConfig = 'couch';
			public $useTable = 'couchdb_database_name';
			public $primaryKey = 'id';
		
		}
		
** IMPORTANT **
Also add the following functions to your Model for the schema to be generated so that create and update functions work.
(These are borrowed from https://github.com/maxwellium/cakephp-couchdb)

My recommendation is to make a Model class and ad these to them and have all your CouchDB based classes extend from it.

	public function schema($field = false) {
		$this->_schema = array_flip(array_keys($this->validate));

		if (isset($this->data[$this->alias]) && is_array($this->data[$this->alias])) {
			$this->_schema = array_merge(
				$this->_schema,
				array_flip(array_keys($this->data[$this->alias])));
		}

		if (is_string($field)) {
			if (isset($this->_schema[$field])) {
				return $this->_schema[$field];
			}
			return null;
    	}

		return $this->_schema;
	}

	public function hasField($name, $checkVirtual = false) {
		if (is_array($name)) {
			foreach ($name as $n) {
				if ($this->hasField($n, $checkVirtual)) {
					return $n;
				}
			}
			return false;
		}

		if ($checkVirtual && !empty($this->virtualFields)) {
			if ($this->isVirtualField($name)) {
				return true;
			}
		}

		// this is the change: rebuilding schema from data everytime so all fields are submitted
		$this->schema();

		if ($this->_schema != null) {
			return isset($this->_schema[$name]);
		}

		return false;
	}


Use Cases
---------

### **Create** an all new document
	
	class MyModel extends Model {
		
		function create_record() {
			$this->save(array('MyModel' => array(
				'anyfield' => 'anyvalue',
				'anotherfield' => array(
					'title' => 'awesome title'
					'content' => 'less awesome content'
				)
			)));
		}
		
	}


### **Update** an existing document

	class MyModel extends Model {
		
		function update_record($id) {
			$this->save(array('MyModel' => array(
				'id' => $id,
				'anyfield' => 'anyvalue',
				'anotherfield' => array(
					'title' => 'awesome title'
					'content' => 'less awesome content'
				)
			)));
		}
		
	}
	
### **Delete** an existing document

	class MyModel extends Model {

		function delete_record($id) {
			$this->delete($id);
		}

	}
	
### **Read** an existing document

	class MyModel extends Model {

		function read_record($id) {
			$this->findById($id);
		}

	}
	
### **Read** data from a view

The [*find* params](http://book.cakephp.org/view/1018/find) to use here are :

* *design* : the CouchDB design where the view resides
* *view* : the name of the view you want to request
* *params* : an array of [query options](http://wiki.apache.org/couchdb/HTTP_view_API#Querying_Options)

		class MyModel extends Model {

			function read_view($start, $end) {
				$this->find('all', array(
					'design' => 'mycouchdbdesign',
					'view' => 'mycouchdbview',
					'params' => array('start_key' => $start, 'end_key' => $end, 'group' => null)
				));
			}

		}

