# Deploying WSO2 Enterprise Integrator
The following sections provide information and instructions on how to cluster the ESB profile of WSO2 Enterprise Integrator (WSO2 EI) with a third-party load balancer.

## The deployment pattern

This deployment scenario uses a two-node ESB cluster. That is, two ESB nodes are configured to serve requests with high availability and scalability. As depicted by the following diagram, the product nodes in the cluster are fronted by an external third-party load balancer, which routes requests to the two nodes on a round-robin basis.

<a href=""><img src="../../assets/img/deployment_ei.png"></a>

## Installing WSO2 Enterprise Integrator

Follow the instructions on [downloading and installing WSO2 EI](../setup/install_in_vm.md) on a single machine.

## Setting up the load balancer

Follow the instructions on [setting up a load balancer](../setup/setting_up_lb.md) for a two-node deployment of WSO2 EI.

## Setting up the primary database

By default, the embedded H2 database is configured as the product's primary database. Select your preferred database type from the following list and follow the given instructions:

* [Setting up MySQL](../setup/db/setting-up-MySQL.md)
* [Setting up a MySQL cluster](../stup/db/setting-up-a-MySQL-Cluster.md)
* [Setting up MSSQL](../setup/db/setting-up-MSSQL.md)
* [Setting up Oracle](../setup/db/setting-up-Oracle.md)
* [Setting up Oracle RAC](../setup/db/setting-up-Orable-RAC.md)
* [Setting up IBM Informix](../setup/db/setting-up-IBM-Informix.md)
* [Setting up IBM DB2](../setup/db/setting-up-IBM-DB2.md)
* [Setting up Maria DB](../setup/db/setting-up-MariaDB.md)
* [Setting up embedded Derby](../setup/db/setting-up-Embedded-Derby.md)
* [Setting up remote Derby](../setup/db/setting-up-Remote-Derby.md)
* [Setting up PostgreSQL](../setup/db/setting-up-PostgreSQL.md)
* [Setting up remote H2](../setup/db/setting-up-Remote-H2.md)

## Updating keystores

1. [Create a new SSL certificate](../security/importing_ssl_certificate.md) and import it to the primary keyStore and trustStore (which are located in the /repository/security/ directory). Your primary keystore can now be configured for SSL communication.
2. [Create a new keystore](../security/creating_keystores.md) to use as the internal keystore (for the purpose of data encryption/decryption in internal data stores).

## Configuring the WSO2 EI nodes

Follow the steps given below to configure the two nodes in the WSO2 EI deployment.

1. Open the esb.toml file of both the nodes. This file is located in the <EI_HOME>/conf/ directory.
2. To specify the default settings that should be applicable to the nodes, update the following toml parameters with the required values.

    ```java
    // The config section that groups the parameters that identify the server.
    [server]

    // The hostname of the server.
    hostname = "localhost"

    // If you are running both product nodes (of your cluster) on the same VM, set a port offset for on the servers.
    offset = 0
    ```
   To find additional server-level parameters that you can use, see the [configuration catalog](../../setup/setting_up_lb/#configuring-the-load-balancer).

3. To enable the two nodes to access the shared database, be sure that the following configurations in the esb.toml file is updated with the details of your database.

    Go to the section on [setting up the primary database](../setup/deploying_wso2_ei.md#Setting up the primary database), select your database type, and verify the parameters specific to your database.

    ```Java
    // The config section that groups the parameters for the primary database that will be shared by both product nodes in the cluster.
    [database.shared.db]

    // Specify the type of database.
    type= "<database_type>"

    // Specify the connection URL of your database. The following default URL connects to the H2 database that is shipped with the product.
    url="jdbc:h2:./repository/database/WSO2SHARED_DB;DB_CLOSE_ON_EXIT=FALSE;LOCK_TIMEOUT=60000"

    // The username for connecting to the database. By default, 'root' is the MySQL username.
    username = "root"

    // The password for connecting to the database. By default, 'root' is the MySQL password.
    password = "root"

    ```

    To find additional parameters for configuring the database connection, see the [configuration catalog](../reference/config_catalog.md#connecting-to-the-user-store).

4. To change the credentials of the admin user of each node in the cluster, update the following parameters in the esb.toml file.
    ``` java
    // The config section that groups the parameters defining the system administrator.
    [super_admin]

    // Specify the username of connecting to the server.
    username = "admin"

    // Specify the password for connecting to the server.
    password = "admin"

    // Set this property to 'true'. This ensures that the admin account is created in the user store.
    create_admin_account = true
    ```

     To find additional parameters for configuring the system administrator, see the [configuration catalog](../reference/config_catalog.md#connecting-to-the-user-store).

 5. To manage the users and roles in the system, update the following parameters in the esb.toml file.
    ``` java
    // The config section that groups the parameters that define the user store (which is the shared DB in this example) connected to the server.
    [user_store]

    // Specify "database" as the database type to infer the connection details of your shared DB.
    type = "database"
    ```
    To find additional parameters for connecting to the user store, see the [configuration catalog](../reference/config_catalog.md#connecting-to-the-user-store).

 6. To define the clustering configurations that specify how the two nodes communicate with one another, add the following to the esb.toml file and update the values.
    ``` java
    // The config section that groups the parameters that define cluster coordination.
    [clustering]

    // Specify the port on which the current server node is running.
    local_member_port = 4000

    // Specify the hostname of the current server node.
    local_member_host = "10.100.5.86"

    // Specify the IP address:port of each of the nodes in the cluster as shown below. Be sure to use the same port number and hostname you specified above.
    members = ["10.100.5.86:4000","10.100.5.86:4001"]
    ```
    To find additional parameters to define cluster coordination, see the [configuration catalog](../../reference/ei_config_catalog#configuring-then).

## Optional configurations

See the topics given below.

### Changing the default user store
By default, the primary database that you configure will be the user store (that stores all user information) for the server. This will be a JDBC user store.

If required, you can configure another database, or an LDAP/Active Directory as your user store and connect it to the product server. Find out more about:

* [Setting up a JDBC user store](../setup/setting_up_jdbc_userstore.md)
* [Setting up a read-only LDAP](../setup/setting_up_ro_ldap.md)
* [Setting up a read-write LDAP](../setup/setting_up_rw_ldap.md)
* [Setting up a read-write Active Directory](../setting up_rw_ad.md)

### Configuring Deployment Synchronization

Configure deployment synchronization so that all the integration artifacts you develop can be shared between all the nodes in the cluster. You need to use the deployment synchronization mechanism that is suitable for your environment. See [Configuring Deployment Synchronization](../setup/deployment_synchronization.md) for instructions.

### Configuring Analytics

If you have already done the configurations explained above, you have the option of applying the following configurations for your deployment.

If you wish to view reports, statistics, and graphs related to the message mediation that happens through the ESB, you need to configure the Analytics profile of WSO2 EI. Follow configuring WSO2 EI Analytics in a production setup.

## Verify your deployment

Ensure that you have taken into account the respective security hardening factors (e.g., changing and encrypting the default passwords, configuring JVM security etc.) before deploying WSO2 EI. For more information, see the [Production Deployment Checklist](../setup/deployment_checklist.md).

## Starting the ESB servers

Start the server using the following standard start-up script.

* On **Linux/MacOS** : `cd <EI_HOME>/bin/ sh intergrator.sh`
* ON **Windows** : `cd <EI_HOME>\bin\wso2server.bat`