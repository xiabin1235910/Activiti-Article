# How to define our own user & group management in activiti

We study how to use own user management in activiti for a while. So let me start to introduce how we implement it.

We take an example for the 'Openstack Keystone User Management', and of course, we can use any user management modules of third party to instead the 'Keystone'

According to the Big Man --- Nadav Azaria --- whose article is [《Activiti Authentication And Identity Management Tutorial》](http://developer4life.blogspot.com/2012/02/activiti-authentication-and-identity.html) in 2012, We start to newest customization.

This article is built on the shoulder of 'activiti-webapp-rest2' module in activiti source code.

Because Activiti has provided for us such entrance which has been used by the 'activiti ldap' module. 
But one question is how the customized classed can be identified by the activiti engine, and how to register them.

Obviously, **'SessionFactory'** has provided us such interface.

>First of all, we must build the class **OwnUserManagerFactory & OwnGroupManagerFactory** which must be implemented the **SessionFactory**. The class mentioned in them will be displayed later.

```java
public class OwnUserManagerFactory implements SessionFactory {
	
	private KeystoneConnection keystoneConnection;
	
	public OwnUserManagerFactory (KeystoneConnection keystoneConnection) {
		this.keystoneConnection = keystoneConnection;
	}

	public Class<?> getSessionType() {
		return UserIdentityManager.class;
	}

	public Session openSession() {
		return new OwnUserManager(this.getKeystoneConnection());
	}

	public KeystoneConnection getKeystoneConnection() {
		return keystoneConnection;
	}

	public void setKeystoneConnection(KeystoneConnection keystoneConnection) {
		this.keystoneConnection = keystoneConnection;
	}
}
```

```
public class OwnGroupMagagerFactory implements SessionFactory {

	private KeystoneConnection keystoneConnection;

	public OwnGroupMagagerFactory(KeystoneConnection keystoneConnection) {
		this.keystoneConnection = keystoneConnection;
	}

	@Override
	public Class<?> getSessionType() {
		return GroupIdentityManager.class;
	}

	@Override
	public Session openSession() {
		return new OwnGroupManager(this.getKeystoneConnection());
	}
	
	public KeystoneConnection getKeystoneConnection() {
		return keystoneConnection;
	}

	public void setKeystoneConnection(KeystoneConnection keystoneConnection) {
		this.keystoneConnection = keystoneConnection;
	}

}
```

>Then, We must define the class **OwnUserManager & OwnGroupManager** which extend the class --- **UserEntityManager**. And we have to implement our own query logic in **在findUserByQueryCriteria中**. Any communication with other user management of third party can be used.

```
import org.activiti.engine.ActivitiException;
import org.activiti.engine.identity.User;
import org.activiti.engine.impl.Page;
import org.activiti.engine.impl.UserQueryImpl;
import org.activiti.engine.impl.persistence.entity.UserEntityManager;

public class OwnUserManager extends UserEntityManager {

	private KeystoneConnection keystoneConnection;
	
	public OwnUserManager(KeystoneConnection keystoneConnection) {
		this.keystoneConnection = keystoneConnection;
	}
	
	@Override
	public User createNewUser(String userId) {
		return super.createNewUser(userId);
//		throw new ActivitiException("User manager doesn't support creating a newe user");
	}

	@Override
	public void insertUser(User user) {
		super.insertUser(user);
//		throw new ActivitiException("User manager doesn't support inserting a newe user");
	}

	@Override
	public void updateUser(User updatedUser) {
		super.updateUser(updatedUser);
//		throw new ActivitiException("User manager doesn't support updating a newe user");
	}

	@Override
	public User findUserById(String userId) {
		return super.findUserById(userId);
//		throw new ActivitiException("User manager doesn't support finding an user by id");
	}

	@Override
	public void deleteUser(String userId) {
		throw new ActivitiException("User manager doesn't support deleting a newe user");
	}

	@Override
	public List<User> findUserByQueryCriteria(UserQueryImpl query, Page page) {
		System.out.println(
				"start to findUserByQueryCriteria.....................!!!!!!_----------------------------------------------------");
		// use your own method or third party method...
		return super.findUserByQueryCriteria(query, page);
	}

	@Override
	public List<User> findUsersByNativeQuery(Map<String, Object> parameterMap, int firstResult, int maxResults) {
		System.out.println(
				"start to findUsersByNativeQuery.....................!!!!!!_----------------------------------------------------");
		// use your own method or third party method...
		return super.findUsersByNativeQuery(parameterMap, firstResult, maxResults);
	}

	@Override
	public long findUserCountByQueryCriteria(UserQueryImpl query) {
		return super.findUserCountByQueryCriteria(query);
//		return findUserByQueryCriteria(query, null).size();
	}

	@Override
	public Boolean checkPassword(String userId, String password) {
		
		// now return true, means that ignoring the password verification
		return true;
	}

}
```

>>The method --- checkPassword --- returns true, means to skip the user verification

```
public class OwnGroupManager extends GroupEntityManager {
	
	private KeystoneConnection keystoneConnection;
	
	public OwnGroupManager(KeystoneConnection keystoneConnection) {
		this.keystoneConnection = keystoneConnection;
	} 

	@Override
	public void insertGroup(Group group) {
		throw new ActivitiException("My group manager doesn't support inserting a new group");
	}

	@Override
	public void updateGroup(Group updatedGroup) {
		throw new ActivitiException("My group manager doesn't support updating a new group");
	}

	@Override
	public void deleteGroup(String groupId) {
		throw new ActivitiException("My group manager doesn't support deleting a new group");
	}

	@Override
	public List<Group> findGroupByQueryCriteria(GroupQueryImpl query, Page page) {
		// sometimes to implement how to query the Group
//		return super.findGroupByQueryCriteria(query, page);
		
		List<Group> groups = new ArrayList<Group>();
		GroupEntity ge = new GroupEntity();
		ge.setId("admin");
		ge.setRevision(1);
		ge.setName("Administrators");
		ge.setType("security-role");
		groups.add(ge);
		return groups;
	}

	@Override
	public long findGroupCountByQueryCriteria(GroupQueryImpl query) {
		// TODO Auto-generated method stub
		return super.findGroupCountByQueryCriteria(query);
	}

	@Override
	public List<Group> findGroupsByUser(String userId) {
		throw new ActivitiException("My group manager doesn't support finding a group");
	}
	
}
```

>In the **findGroupByQueryCriteria || findUserByQueryCriteria** function, we can implement our real logic.

>Furthermore, build the 'KeystoneConnection', of course, it can be any other user configuration POJO.

```
public class KeystoneConnection {
	
	private String protocal;
	
	private String address;
	
	private String port;
	
	public KeystoneConnection() {
		
	}
	
	public KeystoneConnection(String protocal, String address, String port) {
		this.protocal = protocal;
		this.address = address;
		this.port = port;
	}
	
	public String getKeystoneUrl(String url) {
		return this.protocal + "://" + this.address + ":" + this.port + url;
	}

	// getter and setter
}
```

>At last, we must tell the activiti engine, how to identify the two Factory class we have defined in the first step. Of course, it is the process of registration. So we extend the class **ActivitiEngineConfiguration** in the 'activiti-webapp-rest2' module.
Or it will also nicely configure them in spring configuration xml.

```
@Configuration
public class ActivitiEngineConfiguration
```

```
@Bean(name = "processEngineConfiguration")
	public ProcessEngineConfigurationImpl processEngineConfiguration() {
		SpringProcessEngineConfiguration processEngineConfiguration = new SpringProcessEngineConfiguration();
		// add following configuration
		// load the custom manager session factory
		List<SessionFactory> customSessionFactories = new ArrayList<SessionFactory>();
		customSessionFactories.add(new OwnUserManagerFactory(keystoneConnection()));
		customSessionFactories.add(new OwnGroupMagagerFactory(keystoneConnection()));
		processEngineConfiguration.setCustomSessionFactories(customSessionFactories);

		return processEngineConfiguration;
	}
```
**Conclusion**：
**The reason is very simple, the 'SessionFactory' in the Activiti Engine maintains a specific data structure below**

```
HashMap<SessionType, Session>
```

**So the two sub-SessionFactory classes we registered will cover the old ones which are loaded when the activiti engine started**
**And then, it is definitely that the engine will read our own management Factory every time**