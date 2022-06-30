.. _install-kubernetes:

Install Percona Server for MySQL on Kubernetes
==============================================

#. First of all, clone the percona-server-mysql-operator repository:

   .. code:: bash

      git clone -b v{{{release}}} https://github.com/percona/percona-server-mysql-operator
      cd percona-server-mysql-operator

   .. note:: It is crucial to specify the right branch with ``-b``
      option while cloning the code on this step. Please be careful.

#. Now Custom Resource Definition for Percona Server for MySQL should be created
   from the ``deploy/crd.yaml`` file. Custom Resource Definition extends the
   standard set of resources which Kubernetes “knows” about with the new
   items (in our case ones which are the core of the operator). `Apply it <https://kubernetes.io/docs/reference/using-api/server-side-apply/>`_ as follows:

   .. code:: bash

      $ kubectl apply --server-side -f deploy/crd.yaml

   This step should be done only once; it does not need to be repeated
   with the next Operator deployments, etc.

#. The next thing to do is to add the ``mysql`` namespace to Kubernetes,
   not forgetting to set the correspondent context for further steps:

   .. code:: bash

      $ kubectl create namespace mysql
      $ kubectl config set-context $(kubectl config current-context) --namespace=mysql

   .. note:: You can use different namespace name or even stay with the *Default* one.

#. Now RBAC (role-based access control) for Percona Server for MySQL should be set
   up from the ``deploy/rbac.yaml`` file. Briefly speaking, role-based access is
   based on specifically defined roles and actions corresponding to
   them, allowed to be done on specific Kubernetes resources (details
   about users and roles can be found in `Kubernetes
   documentation <https://kubernetes.io/docs/reference/access-authn-authz/rbac/#default-roles-and-role-bindings>`__).

   .. code:: bash

      $ kubectl apply -f deploy/rbac.yaml

   .. note:: Setting RBAC requires your user to have cluster-admin role
      privileges. For example, those using Google Kubernetes Engine can
      grant user needed privileges with the following command:
      ``$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value core/account)``

   Finally it’s time to start the operator within Kubernetes:

   .. code:: bash

      $ kubectl apply -f deploy/operator.yaml

#. Now that’s time to add the Percona Server for MySQL Users secrets to
   Kubernetes. They should be placed in the data section of the
   ``deploy/secrets.yaml`` file as logins and plaintext passwords for the user
   accounts (see `Kubernetes documentation <https://kubernetes.io/docs/concepts/configuration/secret/>`_
   for details).

   After editing is finished, users secrets should be created using the
   following command:

   .. code:: bash

      $ kubectl create -f deploy/secrets.yaml

   More details about secrets can be found in :ref:`users`.

#. Now certificates should be generated. By default, the Operator generates
   certificates automatically, and no actions are required at this step. Still,
   you can generate and apply your own certificates as secrets according
   to the :ref:`TLS instructions <tls>`.

#. After the operator is started and user secrets are added, Percona Server for
   MySQL can be created at any time with the following command:

   .. code:: bash

      $ kubectl apply -f deploy/cr.yaml

   Creation process will take some time. The process is over when both
   operator and replica set pod have reached their Running status.
   ``kubectl get pods`` output should look like this:

   .. include:: ./assets/code/kubectl-get-pods-response.txt

#. Check connectivity to your newly created cluster

   .. code:: bash

      $ kubectl run -i --rm --tty percona-client --image=percona:8.0 --restart=Never -- bash -il
      percona-client:/$ mysql -h cluster1-mysql-primary -uroot -proot_password

   This command will connect you to the MySQL monitor.

   .. include:: ./assets/code/mysql-welcome-response.txt
