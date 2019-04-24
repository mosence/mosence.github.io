---
title: java 嵌入式LDAP
date: 2019-04-24 09:34:00
tags:
---
最近要用到LDAP来统一用户信息存储，当然选型最广泛使用的并且自身就有层级的LDAP来存储了，但是部署一个LDAP服务器还是挺麻烦的，于是找了spring体系下就支持的嵌入式内存LDAP。

直接上代码好了，以下依次是依赖、配置、数据预定义存储文件、Entry类、Repository类

```pom
    <dependency>
        <groupId>com.unboundid</groupId>
        <artifactId>unboundid-ldapsdk</artifactId>
    </dependency>
```

```application.yaml
spring:
  ldap:
    embedded:
      base-dn: dc=mdp,dc=mosence,dc=org
      port: 8888
```

```schema.ldif
dn: dc=mdp,dc=mosence,dc=org
objectClass: top
objectClass: domain
dc: mdp

dn: dc=account,dc=mdp,dc=mosence,dc=org
objectClass: top
objectClass: domain
dc: account

```

```Account.java
@Data
@Entry(base = "dc=account,dc=mdp,dc=mosence,dc=org",objectClasses = "person") //ldif中必须定义top，否则报错
public class Account implements UserDetails {

    @Id
    private Name oid;

    @Attribute(name="cn")
    private String username;

    @DnAttribute(value = "sn",index = 4) //这里的index表示，在entry实例中，dn标识从右到左第几个标识，并且在一个entry中必须最少一个index>=0的@DnAttribute
    @Attribute(name="sn")
    private String id;

    @org.codehaus.jackson.annotate.JsonIgnore
    @com.fasterxml.jackson.annotation.JsonIgnore
    @Transient
    private String password; //因为objectClasses = "person"中，userPassword必须是BINARY类型，所以这里这么搞

    public void setPassword(String password){ //因为objectClasses = "person"中，userPassword必须是BINARY类型，所以这里这么搞
        this.password = password;
        this.bytesPassword = this.password.getBytes();
    }

    @Override
    public String getPassword(){ //因为objectClasses = "person"中，userPassword必须是BINARY类型，所以这里这么搞
        this.password = new String(this.bytesPassword);
        return this.password;
    }

    @Attribute(name = "userPassword", type = Attribute.Type.BINARY)
    @org.codehaus.jackson.annotate.JsonIgnore
    @com.fasterxml.jackson.annotation.JsonIgnore
    private byte[] bytesPassword; //因为objectClasses = "person"中，userPassword必须是BINARY类型，所以这里这么搞

    public void setUsername(String username){
        this.username = username;
        this.id = username;
    }

    @Transient
    private boolean enabled = true;

    @Transient
    private boolean lock = false;

    @Transient
    private boolean delete = false;

    @Transient
    private long expired = -1;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return new HashSet<>(0);
    }


    @Override
    public boolean isAccountNonExpired() {
        return (expired==-1);
    }

    @Override
    public boolean isAccountNonLocked() {
        return !lock;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return (expired==-1);
    }

    @Override
    public boolean isEnabled() {
        return enabled;
    }

}
```

```AccountRepository.java
@Repository
public interface AccountRepository extends LdapRepository<Account> {
    Optional<Account> findByUsername(String username);
}
```

没有实例代码哦，愉快的玩耍吧，反正理解 LDAP 的协议规范后还是很简单，这里就不介绍 LDAP 规范了。