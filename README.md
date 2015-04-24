Armed with high-availability, rolling upgrade of a running OpenStack becomes possible. The utilities listed in this folder will allow rolling upgrade of a Havana cluster to Juno with impacting customer VMs. Some of the administrative functions however might be imapcted during the upgrade window.

Since there are data modeling changes from Havana to Juno, careful MySql database backup is required. It's recommended not to conduct changes during the upgrade.

Followings are the steps to upgrade Keystone from havana 2013.3.2 to 2014.2.2, Juno stable code revision. Oslo dependencies and MySQL data model changes were some of the top chanllenges however issues are mitigable with careful plannings.  

Upgrade steps:


    Remove legacy Keystone packages and dependencies
        python-keystoneclient
        python-keystone 
        openstack-keystone
        python-oslo-config
        python-dogpile-cache
        python-pycadf

    Install thew new Keystone packages and dependencies
        keystone-2014.2.2-1.noarch
        python-keystoneclient-0.9.0-1.el6.noarch
         aioeventlet-0.4-1.noarch  
        dogpile.cache-0.5.6-1.noarch  
        libaio-0.3.107-10.el6.x86_64  
        oslo.config-1.9.0-1.noarch  
        oslo.db-1.5.0-1.noarch  
        oslo.i18n-1.4.0-1.noarch  
        oslo.messaging-1.7.0-1.noarch  
        oslo.middleware-0.5.0-1.noarch  
        oslo.serialization-1.3.0-1.noarch  
        oslo.utils-1.3.0-1.noarch  
        pycadf-0.8.0-1.noarch  
        python-dogpile-core-0.4.1-1.el6.noarch  
        python-routes1.12-1.12.3-4.el6.noarch  
        repoze.lru-0.6-1.noarch  
        SQLAlchemy-0.9.8-1.x86_64  
        trollius-1.0.4-1.noarch 

    Modify the Keystone schema and tables
    Keystone Schema Changes
    -- MySQL dump 10.13  Distrib 5.6.21-70.1, for Linux (x86_64)
    --
    -- Host: localhost    Database: keystone
    -- ------------------------------------------------------
    -- Server version    5.6.21-70.1-56
    /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
    /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
    /*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
    /*!40101 SET NAMES utf8 */;
    /*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
    /*!40103 SET TIME_ZONE='+00:00' */;
    /*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
    /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
    /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
    /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
    --
    -- Table structure for table `assignment`
    --
    DROP TABLE IF EXISTS `assignment`;
    /*!40101 SET @saved_cs_client     = @@character_set_client */;
    /*!40101 SET character_set_client = utf8 */;
    CREATE TABLE `assignment` (
      `type` enum('UserProject','GroupProject','UserDomain','GroupDomain') NOT NULL,
      `actor_id` varchar(64) NOT NULL,
      `target_id` varchar(64) NOT NULL,
      `role_id` varchar(64) NOT NULL,
      `inherited` tinyint(1) NOT NULL,
      PRIMARY KEY (`type`,`actor_id`,`target_id`,`role_id`),
      KEY `role_id` (`role_id`),
      KEY `ix_actor_id` (`actor_id`),
      CONSTRAINT `assignment_ibfk_1` FOREIGN KEY (`role_id`) REFERENCES `role` (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    /*!40101 SET character_set_client = @saved_cs_client */;
    --
    -- Table structure for table `id_mapping`
    --
    DROP TABLE IF EXISTS `id_mapping`;
    /*!40101 SET @saved_cs_client     = @@character_set_client */;
    /*!40101 SET character_set_client = utf8 */;
    CREATE TABLE `id_mapping` (
      `public_id` varchar(64) NOT NULL,
      `domain_id` varchar(64) NOT NULL,
      `local_id` varchar(64) NOT NULL,
      `entity_type` enum('user','group') NOT NULL,
      PRIMARY KEY (`public_id`),
      UNIQUE KEY `domain_id` (`domain_id`,`local_id`,`entity_type`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    /*!40101 SET character_set_client = @saved_cs_client */;
    --
    -- Table structure for table `region`
    --
    DROP TABLE IF EXISTS `region`;
    /*!40101 SET @saved_cs_client     = @@character_set_client */;
    /*!40101 SET character_set_client = utf8 */;
    CREATE TABLE `region` (
      `id` varchar(255) NOT NULL,
      `description` varchar(255) NOT NULL,
      `parent_region_id` varchar(255) DEFAULT NULL,
      `extra` text,
      `url` varchar(255) DEFAULT NULL,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    /*!40101 SET character_set_client = @saved_cs_client */;
    --
    -- Table structure for table `revocation_event`
    --
    DROP TABLE IF EXISTS `revocation_event`;
    /*!40101 SET @saved_cs_client     = @@character_set_client */;
    /*!40101 SET character_set_client = utf8 */;
    CREATE TABLE `revocation_event` (
      `id` varchar(64) NOT NULL,
      `domain_id` varchar(64) DEFAULT NULL,
      `project_id` varchar(64) DEFAULT NULL,
      `user_id` varchar(64) DEFAULT NULL,
      `role_id` varchar(64) DEFAULT NULL,
      `trust_id` varchar(64) DEFAULT NULL,
      `consumer_id` varchar(64) DEFAULT NULL,
      `access_token_id` varchar(64) DEFAULT NULL,
      `issued_before` datetime NOT NULL,
      `expires_at` datetime DEFAULT NULL,
      `revoked_at` datetime NOT NULL,
      `audit_id` varchar(32) DEFAULT NULL,
      `audit_chain_id` varchar(32) DEFAULT NULL,
      PRIMARY KEY (`id`),
      KEY `ix_revocation_event_revoked_at` (`revoked_at`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    /*!40101 SET character_set_client = @saved_cs_client */;
    /*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
    /*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
    /*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
    /*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
    /*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
    /*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
    /*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
    /*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;
    -- Dump completed on 2015-03-06 22:22:12
     
    In addition, add 2 columns in keystone.endpoint by adding enabled (tinyint) and region_id (varchar(255) ) and 1 column in keystone.service by adding enabled (tinyint)
     
     
    Populate Following keystone.assignment table with configured data accordingly.
     
    type --> "UserProject"
    actor_id --> "user_id" including admin/user and admin accounts such as nova
    target_id --> the tenant id which the actor id is associated with
    role-id --> the role id assigned to the actor_id such as Admin. Member or else  
    inherited --> '0': no, '1': yes


    Modify the configuration
        Add keystone-paste.ini
        Add config_file to keystone.conf
        Add lock_file=None to nova.conf
    Write a new Keystone startup script    
        Write a new openstack-keystone for starup

Patch Issues
 
Issue: Keystone trust table column missing
(when running keystone --insecure user-role-remove <>)
[root@ha-controller1 patches]# kk user-role-list --user wpc-ci-user1
[root@ha-controller1 patches]# kk user-role-add --user wpc-ci-user1 --tenant contrail --role admin
[root@ha-controller1 patches]# kk user-role-list --user wpc-ci-user1
    +----------------------------------+-------+--------------+----------------------------------+
    |                id                |  name |   user_id    |            tenant_id             |
    +----------------------------------+-------+--------------+----------------------------------+
    | edecdf6d9f434a159a46ab6ed4a06955 | admin | wpc-ci-user1 | 961b69b53004430781a0eaf6d498739b |
    +----------------------------------+-------+--------------+----------------------------------+
    [root@ha-controller1 patches]# kk user-role-remove --user wpc-ci-user1 --tenant contrail --role admin
    An unexpected error prevented the server from fulfilling your request. (HTTP 500)
    [root@ha-controller1 patches]# kk user-role-list --user wpc-ci-user1

 

    2015-03-19 20:14:47.029 6416 TRACE keystone.common.wsgi OperationalError: (OperationalError) (1054, "Unknown column 'trust.remaining_uses' in 'field list'") 'SELECT trust.id AS trust_id, trust.trustor_user_id AS trust_trustor_user_id, trust.trustee_user_id AS trust_trustee_user_id, trust.project_id AS trust_project_id, trust.impersonation AS trust_impersonation, trust.deleted_at AS trust_deleted_at, trust.expires_at AS trust_expires_at, trust.remaining_uses AS trust_remaining_uses, trust.extra AS trust_extra \nFROM trust \nWHERE trust.deleted_at IS NULL AND trust.trustee_user_id = %s' ('wpc-ci-user1',)
    Current trust table
    mysql> show columns from trust;

    +-----------------+-------------+------+-----+---------+-------+
    | Field           | Type        | Null | Key | Default | Extra |
    +-----------------+-------------+------+-----+---------+-------+
    | id              | varchar(64) | NO   | PRI | NULL    |       |
    | trustor_user_id | varchar(64) | NO   |     | NULL    |       |
    | trustee_user_id | varchar(64) | NO   |     | NULL    |       |
    | project_id      | varchar(64) | YES  |     | NULL    |       |
    | impersonation   | tinyint(1)  | NO   |     | NULL    |       |
    | deleted_at      | datetime    | YES  |     | NULL    |       |
    | expires_at      | datetime    | YES  |     | NULL    |       |
    | extra           | text        | YES  |     | NULL    |       |
    +-----------------+-------------+------+-----+---------+-------+

    We need to find the correct traits to give this column as well.  We already add several columns earlier in the patch. 
