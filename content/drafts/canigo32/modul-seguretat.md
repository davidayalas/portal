+++
date        = "2016-11-28T11:48:54+02:00"
title       = "Seguretat"
description = "Autentificació i autorització d'usuaris."
sections    = "Canigó. Documentació versió 3.x"
weight      = 9
+++

## Propòsit

El Servei de Seguretat té com a propòsit principal gestionar l'autentificació i l'autorització dels usuaris de les nostres aplicacions. L'objectiu de l'autentificació és comprovar que l'usuari és qui diu ser, mentre que l'autorització s'encarrega de comprovar que realment té accés al recurs sol.licitat.
	
<div class="message warning">
L'especificació JAAS (Java Authorization and Authentication) de J2EE proporciona els mecanismes necessaris de seguretat. Cada servidor d'aplicacions pot implementar l'estàndard però ho fa de diferents formes produint problemes de compatibilitat.  
L'especificació JAAS s'orienta principalment a temes d'autentificació, mentre que els temes d'autorització pateixen de moltes carències.
</div>

Actualment, i a causa del seu grau de maduresa i facilitat Canigó recomana l'ús de 'Spring Security 4.2.1' com framework base i les extensions que Canigó proporciona.

## Instal.lació i Configuració

### Instal.lació

Per tal d'instal- lar el mòdul de seguretat es pot incloure automàticament a través de l'eina de suport al desenvolupament o bé afegir manualment en el pom.xml de l'aplicació la següent dependència:

```
<canigo.security.version>[1.2.0, 1.3.0)</canigo.security.version>

<dependency>
	<groupId>cat.gencat.ctti</groupId>
	<artifactId>canigo.security</artifactId>
	<version>${canigo.security.version}</version>
</dependency>

```

### Configuració

La configuració es realitza automàticament a partir de la eina de suport al desenvolupament i es descompon en les parts següents:

* Configuració dels filtres de l'aplicació REST
* Configuració de JWT
* Configuració de l'Autenticació
* Configuració de l'Autorització
* Configuració de la font de dades de l'esquema de seguretat

#### Configuració dels filtres de l'aplicació REST

Spring Security utilitza un conjunt de filtres per a detectar aspectes de l'autorització i autentificació. Per a fer-los servir definirem en el fitxer 'WEB-INF/web.xml el codi següent:

```
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

Per a més informació consultar la pàgina [Spring Security Doc](http://docs.spring.io/spring-security/site/docs/4.2.x/reference/htmlsingle/#security-filter-chain)

#### Configuració de JWT
La nova versió de Canigó treballa amb JWT (JSON web Token). Per això s'ha fet servir la llibreria Java JJWT. Aquesta llibreria permet autenticar l'usuari amb qualsevol dels mètodes descrits en el següent apartat, Configuració de l'Autenticació. Un cop autenticat l'usuari es genera un token que serà enviat a cada petició a la capçalera. Aquest token conté tota la informació de l'usuari pel que facilita l'escalabilitat del sistema. Per poder configurar JWT es necessita afegir al fitxer de propietats del Servei de Seguretat (security.properties) les següents propietats:

Propietat                     | Requerit | Descripció                                 | Valor per Defecte
----------------------------- | -------- | -------------------------------------------|------------------
*.jwt.header                  | No       | nom del header del token JWT		      | Authentication
*.jwt.header.startToken       | No       | Inici del token JWT       		      | Bearer
*.jwt.tokenResponseHeaderName | No       | Nom del header del token JWT        	      | jwtToken
*.jwt.secret                  | No       | Password per generar el token JWT          | canigo
*.jwt.expiration              | No       | Temps de vida del token JWT       	      | 3600
*.jwt.siteminderAuthentication| No       | Gicar authentication             	      | false

Per a més informació sobre JWT visitar la pàgina oficial a [JWT page] (https://jwt.io/)

#### Configuració de l'Autenticació

En la configuració de l'Autentificació tindrem en consideració:

* Seleccionar la configuració de la font en que es realitza l'autentificació (per arxiu de propietats, base de dades, LDAP, per servei integrador al servidor corporatiu basat en HTTPS, ...)
* Configurar el formulari d'autentificació web i la seqüència de cerca on ha de realitzar-se l'autentificació.

Dins d'aquest mòdul trobem els següents proveidors de seguretat:

* Seguretat InMemory
* Seguretat Base de dades
* Seguretat LDAP
* Seguretat GICAR

Els diferents proveidors comparteixen els següents arxius de configuració:

* security.properties: Propietats del servei de seguretat
* app-custom-security.xml: Arxiu XML amb la configuració de seguretat.
* security.users.properties: Llistat en format pla dels usuaris/password/rols de l'aplicació per al proveidor "InMemory".

La disposició dels arxius és la següent:

* Ubicació: <PROJECT_ROOT>/src/main/resources/config/props/security.users.properties
* Ubicació: <PROJECT_ROOT>/src/main/resources/spring/app-custom-security.xml
* Ubicació: <PROJECT_ROOT>/src/main/resources/config/props/security.properties

```
<security:http pattern="/css/**" security="none"/>
<security:http pattern="/images/**" security="none"/>
<security:http pattern="/js/**" security="none"/>

<!--  Secure patterns -->
<security:http>
  	<security:intercept-url pattern="/**/views/login.xhtml" 		access="IS_AUTHENTICATED_ANONYMOUSLY" />
	<security:intercept-url pattern="/**/views/logout.xhtml" 			access="IS_AUTHENTICATED_ANONYMOUSLY"/>
	<security:intercept-url pattern="/**/views/monitoring.xhtml"		access="ROLE_ADMIN" />
	<security:intercept-url pattern="/**/views/propertyExpose.xhtml"	access="ROLE_ADMIN" />
	<security:intercept-url pattern="/**/views/moduleList.xhtml"		access="ROLE_ADMIN" />
	<security:intercept-url pattern="/**/views/logReader.xhtml" 		access="ROLE_ADMIN"/>
	<security:intercept-url pattern="/**/views/**" 					access="ROLE_USER"/>

	    
	<security:form-login 	login-processing-url="/AppJava/j_spring_security_check"
							login-page="/AppJava/views/login.xhtml?set-locale=ca_ES" 
							authentication-failure-url="/AppJava/views/login.xhtml?set-locale=ca_ES&amp;error=1"/>
	<security:logout logout-url="/AppJava/views/logout.xhtml" />
	<security:access-denied-handler error-page="/views/accessDenied.xhtml"/>
		
</security:http>
	
<security:authentication-manager>

<!--Proveidor de seguretat-->

</security:authentication-manager>
```

#### Configuració de la Font d'Autorització per base de dades

Per a configurar la font d'autorització mitjançant base de dades és necessari:

* Configurar l'arxiu de propietats security.properties.
* Conigurar el proveidor de seguretat dins de la configuració de seguretat de Spring.

Els dos arxius es generen i configuren de manera automàtica mitjançant l'eina de desenvolupament.

Les propietats de l'arxiu security.properties son les següents:

Propietat                    | Requerit | Descripció
---------------------------- | -------- | ----------------------------------------------------------
*.security.database.jndiName | Si       | Nom JNDI d'accés a la BD. Obligatori per a connexions JNDI
*.security.database.url      | Si       | URL de connexió a la base de dades
*.security.database.username | Si       | Usuari de connexió a la base de dades
*.security.database.password | Si       | Password de connexió a la base de dades

La configuració del provider en app-custom-security.xml per aquest proveidor és la següent:

* Afegim el provider al authentication manager.
* Afegim el tipus de codificador de password per tal de comparar el password de base de dades i el que ens ha proporcionat l'usuari de l'aplicació. Aquest codificador suporta: plaintext, sha, sha-256, md5, md4, ssha. Si en la base de dades tenim emmagatzemat els password d'usuari en md5, i marquem "password-encode" com a md5, de manera automàtica, el password proporcionat per l'usuari via formulari de login (j_password) es codificarà en md5 per posteriorment ser comparat amb el emmagatzemat a la base de dades.

```
<security:authentication-manager>
	<security:authentication-provider>
                <security:password-encoder hash="plaintext"/>
		<security:jdbc-user-service data-source-ref="dataSource"/>
	</security:authentication-provider>
</security:authentication-manager>
```

<div class="message warning">
Accés base de dades

L'eina de suport al desenvolupament automatitza la instal- lació del mòdul de persistència si aquest no ha estat instal- lat prèviament pel desenvolupador.
</div>

#### Configuració de l'Autorització per LDAP

Per a configurar l'acces a LDAP és necessari:

* Configurar l'arxiu de propietats security.properties.
* Conigurar el proveidor de seguretat dins de la configuració de seguretat de Spring.

Els dos arxius es generen i configuren de manera automàtica mitjançant l'eina de desenvolupament.

Les propietats de l'arxiu security.properties son les següents:

Propietat                           | Requerit | Descripció
----------------------------------- | -------- | -----------------------------------------------------------------------
*.security.ldap.url                 | Si       | Direcció del servidor ldap separat amb dos punts ":" del port
*.security.ldap.manager.dn          | Sí       | Identificador de l'usuari administrador del LDAP
*.security.ldap.manager.password    | Si       | Password del l'usuari administrador del LDAP
*.security.ldap.user.search.filter  | No       | Filtre de cerca dels usuaris dintre de l'estructura del LDAP. Per defecte: (uid={0})
*.security.ldap.user.search.base    | Si       | String base de la ubicació dels usuaris dintre de l'estructura del LDAP
*.security.ldap.group.search.base   | Si       | String base de la ubicació dels grups dintre de l'estructura del LDAP
*.security.ldap.group.search.filter | No       | Filtre de cerca dels grups dintre de l'estructura del LDAP. Per defecte: (cn={0})

Per a realitzar les proves en desenvolupament podem instal- lar un servidor LDAP senzill (veure l'apartat 'Eines de Suport' per a més referència).

Configuració del provider en **app-custom-security.xml**:

* Afegim el provider al authentication manager.
* Afegim la connexió al LDAP server

```
<security:authentication-manager>
    <security:ldap-authentication-provider
        server-ref="ldapLocal"
        user-search-filter="${security.ldap.user.search.filter:(uid={0})}"
        user-search-base="${security.ldap.user.search.base}"
        group-search-base="${security.ldap.group.search.base}"
        group-search-filter="${security.ldap.group.search.filter:(cn={0})}">

        <security:password-compare/>
    </security:ldap-authentication-provider>
</security:authentication-manager>

<security:ldap-server url="${security.ldap.url}" id="ldapLocal" manager-dn="${security.ldap.manager.dn}"
manager-password="${security.ldap.manager.password}"/>
```

#### Configuració de l'Autorització per arxiu de propietats

Aquest proveidor de seguretat es recolça en un arxiu de propietats per carregar en memòria els usuaris/password/rols de l'aplicació.

Per a configurar l'acces mitjançant un arxiu de propietats és necessari:

* Configurar l'arxiu de propietats **security.users.properties**.
* Conigurar el proveidor de seguretat dins de la configuració de seguretat de Spring (**app-custom-security.xml**).

L'arxiu que conté aquesta configuració **security.users.properties** te el següent format:

    username=password,grantedAuthority[,grantedAuthority][,enabled|disabled]

Exemple de configuració:

```
user=password,ROLE_USER,enabled
admin=password,ROLE_USER,ROLE_ADMIN,enabled
```

#### Configuració del provider en **app-custom-security.xml**:

* Afegim el provider al authentication manager.
* Afegim el tipus de codificador de password per tal de comparar el password de l'arxiu de propietats i el que ens ha proporcionat l'usuari de l'aplicació. Aquest codificador suporta: plaintext, sha, sha-256, md5, md4, ssha.

```
<security:authentication-manager>
	<security:authentication-provider>
		<security:password-encoder hash="plaintext"/>
		<security:user-service properties="classpath:/config/props/security.users.properties"/>
	</security:authentication-provider>
</security:authentication-manager>
```

#### Configuració de la Font d'Autorització per GICAR

Per a configurar l'acces a GICAR és necessari:

* Configurar l'arxiu de propietats **security.properties**.
* Configurar el proveidor de seguretat dins de la configuració de seguretat de Spring.

Els dos arxius es generen i configuren de manera automàtica mitjançant l'eina de desenvolupament:

Les propietats de l'arxiu **security.properties** son les següents:

Per a configurar l'acces a GICAR és necessari configurar l'arxiu de propietats **security.properties**. Aquest arxiu es genera automàticament des de l'eina de suport, i te el següent format:

Propietat                                   | Requerit | Descripció
------------------------------------------- | -------- | -----------------------------------
*.security.gicar.httpGicarHeaderUsernameKey | No       | Aquesta propietat indica quin és el camp de la capçalera HTTP_GICAR que conté el nom de l'usuari autenticat a GICAR. Per defecte: NIF

La configuració específica del proveidor és el següent:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans 	xmlns="http://www.springframework.org/schema/beans"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns:security="http://www.springframework.org/schema/security"
		xsi:schemaLocation="http://www.springframework.org/schema/beans 	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
					       http://www.springframework.org/schema/security 	http://www.springframework.org/schema/security/spring-security-3.0.xsd">

	<security:http pattern="/css/**" security="none"/>
	<security:http pattern="/images/**" security="none"/>
	<security:http pattern="/js/**" security="none"/>

	<!--  Secure patterns -->
	<security:http>
  	    <security:intercept-url pattern="/**/views/login.xhtml" 			access="IS_AUTHENTICATED_ANONYMOUSLY" />
	    <security:intercept-url pattern="/**/views/logout.xhtml" 			access="IS_AUTHENTICATED_ANONYMOUSLY"/>
	    <security:intercept-url pattern="/**/views/monitoring.xhtml"		access="ROLE_ADMIN" />
	    <security:intercept-url pattern="/**/views/propertyExpose.xhtml"	access="ROLE_ADMIN" />
	    <security:intercept-url pattern="/**/views/moduleList.xhtml"		access="ROLE_ADMIN" />
	    <security:intercept-url pattern="/**/views/logReader.xhtml" 		access="ROLE_ADMIN"/>
	    <security:intercept-url pattern="/**/views/**" 					access="ROLE_USER"/>


	    <security:logout logout-url="/views/logout.xhtml" />
	    <security:access-denied-handler error-page="/views/accessDenied.xhtml"/>
		
	<!-- GICAR -->
	    <security:form-login login-processing-url="/j_spring_security_check" login-page="/j_spring_security_check" />
	    <security:custom-filter ref="proxyUsernamePasswordAuthenticationFilter" before="FORM_LOGIN_FILTER" />
	</security:http>

	<security:authentication-manager alias="authenticationManager">
		<!-- GICAR -->
		<security:authentication-provider ref="gicarProvider"/>
	</security:authentication-manager>

	<!-- GICAR -->
	<bean id="proxyUsernamePasswordAuthenticationFilter" class="cat.gencat.ctti.canigo.arch.security.authentication.ProxyUsernamePasswordAuthenticationFilter">
		<property name="siteminderAuthentication" 		value="true" />
		<property name="authenticationManager" 			ref="authenticationManager" />
		<property name="authenticationFailureHandler" 	ref="failureHandler" />
	</bean>

	<bean id="failureHandler" class="org.springframework.security.web.authentication.SimpleUrlAuthenticationFailureHandler">
		<property name="defaultFailureUrl" value="/gicar-error.html" />
	</bean>

	<bean id="gicarProvider" class="cat.gencat.ctti.canigo.arch.security.provider.siteminder.SiteminderAuthenticationProvider">
		<description>GICAR Provider</description>
		<property name="userDetailsService" ref="gicarUserDetailsService"/>
	</bean>

	<bean id="gicarUserDetailsService" class="cat.gencat.ctti.canigo.arch.security.provider.gicar.GICARUserDetailsServiceImpl">
		<description>User Detail service implementation for GICAR provider</description>
		<property name="httpGicarHeaderUsernameKey" value="${security.gicar.httpGicarHeaderUsernameKey:NIF}"/>
		<property name="authoritiesDAO" ref="authoritiesDAO"/>
	</bean>

	<bean id="authoritiesDAO" class="cat.gencat.ctti.canigo.arch.security.provider.sace.authorities.AuthoritiesDAOImpl">
		<description>Authorities DAO implementation for SACE. Gets granted authorities for specified user</description>
		<property name="dataSource" ref="dataSource"/>
	</bean>
</beans>
```

Amb aquesta configuració ha de ser possible autoritzar un usuari que prèviament ha estat auntenticat en el servei de GICAR. Per aquest motiu és necessari rebre certes dades referents a aquesta autenticació ja realitzada. A la capçalera HTML podrem accedir a aquestes dades:

```
HTTP_GICAR=CODIINTERN=NRDRJN0001;NIF=11112222W;EMAIL=mail.admin@gencat.net;UNITAT_MAJOR=CTTI;
UNITAT_MENOR=CTTI Qualitat
```

On CODIINTERN és el codi intern, el NIF el NIF, EMAIL l'adreça de correu electrònic registrada al Director Corporatiu, UNITAT_MAJOR és l'organització i UNITAT_MENOR és la unitat.

<div class="message warning">
En cas de que l'aplicació utilitzi la separació entre codi estàtic i dinàmic és necessari indicar la següent propietat dintre del bean proxyUsernamePasswordAuthenticationFilter:
<br><br>

<b>&lt;property name="filterProcessesUrl" value="/AppJava/j_spring_security_check" /&gt;</b>

</div>

**Logout**

Per tots els mètodes d'autentificació, el procediment de logoff consisteix en invalidar la sessió, forçant així que el servei de seguretat intervingui en la següent petició solicitant la nova identificació de l'usuari.

En el cas de Gicar, però, aquesta autentificació és realitzada per un sistema extern a l'aplicació i, per tant, s'ha de comunicar a aquest sistema extern la intenció de fer el logoff. El mecanisme previst per fer-ho consisteix en una URL de Gicar que, al ser invocada, realitza el logoff.

Aquest enllaç de logout és depenent de l'agent de SiteMinder que l'aplicació fa servir per a comunicar-se amb el Policy Server.

Així doncs, els enllaços de logout són els següents:

* Per a una aplicació ubicada en els apaches corporatius de internet: http://sso.gencat.cat/siteminderagent/forms/logoff.html
* Per a una aplicació ubicada en els apaches corporatius de intranet: http://sso.gencat.intranet/siteminderagent/forms/logoff.html
* Per a una aplicació amb apache "pròpi": http://****.gencat.***/siteminderagent/forms/logoff.html;

## Eines de Suport

### Servidor LDAP de proves: openLDAP

Els diferents passos per a instal- lar openLDAP i importar un directori LDAP d'exemple són:

* Baixar openLDAP per a Windows http://sourceforge.net/projects/openldapwindows/ i instal- lar-ho.
* Canviar la configuració per defecte de openldap. Copiar les dades següents en un fitxer slapd.conf. Copiarem el slapd.conf en la mateixa carpeta que openLDAP.

```
#######################################################################
# See slapd.conf(5) for details on configuration options.
# This file should NOT be world readable.
#######################################################################
ucdata-path ./ucdata
include ./etc/schema/core.schema
include ./etc/schema/cosine.schema
include ./etc/schema/inetorgperson.schema

# Define global ACLs to disable default read access.

# Do not enable referrals until AFTER you have a working directory
# service AND an understanding of referrals.
#referral ldap://root.openldap.org

pidfile ./var/run/slapd.pid
argsfile ./var/run/slapd.args

#######################################################################
# BDB database definitions
#######################################################################

###database bdb
###suffix "dc=my-domain,dc=com"
###rootdn "cn=Manager,dc=my-domain,dc=com"

database bdb
suffix dc="mycompany",dc="com"
rootdn "cn=Manager,dc=mycompany,dc=com"

rootpw secret
Eines de Suport

directory ./var/openldap-data
index objectClass eq
```

* Obrir una pantalla "DOS command", anar a la carpeta on hem instal- lat el programa i arrancar openLDAP amb la comanda

```
.\slapd -d 1
```

Si tot ha funcionat bé, hauríem de veure una sortida com la següent:  
    
![Execució Open LDAP](/related/canigo/documentacio/modul-seguretat/ServeiSeguretat_img012.jpg.gif)

* Copiar les dades següents en un fitxer setup.ldif. Aquest fitxer conté un directori LDAP de l'empresa "mycompany.com" amb 2 persones: "gestoruser" i "usuari". Copiarem el setup.ldif en la mateixa carpeta que openLDAP.

```
### Top level definition
#dn: dc=mycompany,dc=com
#objectClass: top
#objectClass: dcObject
#objectClass: domain
#dc: mycompany

### organizationalUnit : PEOPLE
# Definition of people
dn: ou=people,dc=mycompany,dc=com
objectClass: top
objectClass: organizationalUnit
ou: people

# Gestor User
dn: uid=gestoruser,ou=people,dc=mycompany,dc=com
objectClass: person
objectClass: inetOrgPerson
cn: State App
displayName: App Admin
givenName: App
mail: gestor@fake.org
title: ROLE_ADMIN
sn: Gestor
uid: gestoruser
userPassword: gestorpassword

# usuario normal
dn: uid=usuario,ou=people,dc=mycompany,dc=com
objectClass: person
objectClass: inetOrgPerson
cn: State App
displayName: App Admin
givenName: App
mail: usuario@fake.org
title: ROLE_USER
sn: Usuario
uid: usuario
userPassword: usuariopassword
```

Obrir una altra pantalla "DOS command", anar a la carpeta on hem instal- lat el programa i importar les dades amb la comanda:

```
ldapadd -x -D "cn=Manager,dc=mycompany,dc=com" -W -f setup.ldif
```

La contrasenya per defecte és "secret".

### Jxplorer

Comprovarem que la importació de dades ha funcionat amb Jxplorer, un client LDAP Java i opensource.

* Baixar Jxplorer a la url http://sourceforge.net/projects/jxplorer/ i instal- lar-ho
* Prémer el botó per a connectar-se al nostre directori LDAP.

La contrasenya per defecte és "secret". La pantalla següent mostra els valors de la diferents paràmetres:

![Configuració paràmetres JXplorer](/related/canigo/documentacio/modul-seguretat/ServeiSeguretat_img013.jpg.gif)

Si tot ha funcionat bé, hauríem de veure la pantalla següent:

![Resultat JXplorer](/related/canigo/documentacio/modul-seguretat/ServeiSeguretat_img014.jpg.gif)