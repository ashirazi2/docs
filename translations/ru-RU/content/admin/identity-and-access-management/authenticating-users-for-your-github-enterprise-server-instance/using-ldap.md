---
title: Использование протокола LDAP
redirect_from:
- /enterprise/admin/articles/configuring-ldap-authentication
- /enterprise/admin/articles/about-ldap-authentication
- /enterprise/admin/articles/viewing-ldap-users
- /enterprise/admin/hidden/enabling-ldap-sync
- /enterprise/admin/hidden/ldap-sync
- /enterprise/admin/user-management/using-ldap
- /enterprise/admin/authentication/using-ldap
- /admin/authentication/using-ldap
- /enterprise/admin/authentication/authenticating-users-for-your-github-enterprise-server-instance/using-ldap
intro: LDAP lets you authenticate {% data variables.product.prodname_ghe_server %} against your existing accounts and centrally manage repository access. LDAP is a popular application protocol for accessing and maintaining directory information services, and is one of the most common protocols used to integrate third-party software with large company user directories.
versions:
  ghes: '*'
type: how_to
topics:
- Accounts
- Authentication
- Enterprise
- Identity
ms.openlocfilehash: 28c4f7e4d7065ff58d51fbcf1a5370015cb48206
ms.sourcegitcommit: ea9a577cff7ec16ded25ed57417c83ec04816428
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2022
ms.locfileid: "141530406"
---
{% data reusables.enterprise_user_management.built-in-authentication %}

## <a name="supported-ldap-services"></a>Поддерживаемые службы LDAP

{% data variables.product.prodname_ghe_server %} интегрируется со следующими службами LDAP:

* Active Directory
* FreeIPA
* Oracle Directory Server Enterprise Edition
* OpenLDAP
* Open Directory
* 389-ds

## <a name="username-considerations-with-ldap"></a>Рекомендации по использованию имен пользователей в LDAP

{% data reusables.enterprise_management_console.username_normalization %}

{% data reusables.enterprise_management_console.username_normalization_sample %}

{% data reusables.enterprise_user_management.two_factor_auth_header %} {% data reusables.enterprise_user_management.2fa_is_available %}

## <a name="configuring-ldap-with--data-variablesproductproduct_location-"></a>Настройка протокола LDAP с помощью {% data variables.product.product_location %}

После настройки протокола LDAP пользователи смогут входить в ваш экземпляр с помощью своих учетных данных LDAP. При первом входе пользователей имена их профилей, адреса электронной почты и ключи SSH будут заданы с помощью атрибутов LDAP из вашего каталога.

При настройке доступа LDAP для пользователей с помощью {% data variables.enterprise.management_console %} ваши лицензии "на пользователя" не используются, пока пользователь не выполнит первый вход в ваш экземпляр. Однако если вы создаете учетную запись вручную с помощью параметров администратора сайта, лицензия "на пользователя" будет учитываться сразу же.

{% warning %}

**Предупреждение.** Перед настройкой протокола LDAP в {% data variables.product.product_location %} убедитесь, что ваша служба LDAP поддерживает постраничный вывод результатов.

{% endwarning %}

{% data reusables.enterprise_site_admin_settings.access-settings %} {% data reusables.enterprise_site_admin_settings.management-console %} {% data reusables.enterprise_management_console.authentication %}
3. В разделе "Проверка подлинности" выберите **LDAP**.
![Выбор LDAP](/assets/images/enterprise/management-console/ldap-select.png)
4. {% data reusables.enterprise_user_management.built-in-authentication-option %} ![Флажок выбора встроенной проверки подлинности LDAP](/assets/images/enterprise/management-console/ldap-built-in-authentication.png)
5. Добавьте параметры конфигурации.

## <a name="ldap-attributes"></a>Атрибуты LDAP
Используйте эти атрибуты, чтобы завершить настройку LDAP для {% data variables.product.product_location %}.

| Имя атрибута           | Тип     | Описание |
|--------------------------|----------|-------------|
| `Host`                   | Обязательно | Узел LDAP, например `ldap.example.com` или `10.0.0.30`. Если имя узла доступно только из внутренней сети, может потребоваться сначала настроить службу DNS {% data variables.product.product_location %}, чтобы она могла разрешить имя узла с помощью внутренних серверов доменных имен. |
| `Port`                   | Обязательно | Порт, в котором прослушиваются службы LDAP узла. Примеры: 389 и 636 (для LDAPS). |
| `Encryption`             | Обязательно | Метод шифрования, используемый для защиты взаимодействия с сервером LDAP. Примеры: обычный (без шифрования), SSL/LDAPS (шифрование с самого начала) и StartTLS (обновление до зашифрованного взаимодействия после подключения). |
| `Domain search user`     | Необязательно | Пользователь LDAP, который ищет других пользователей, входящих в систему, чтобы разрешить проверку подлинности. Обычно это учетная запись службы, созданная специально для интеграции сторонних пользователей. Используйте полное имя, такое как `cn=Administrator,cn=Users,dc=Example,dc=com`. В Active Directory можно также использовать синтаксис `[DOMAIN]\[USERNAME]` (например `WINDOWS\Administrator`) для пользователя поиска домена в Active Directory. |
| `Domain search password` | Необязательно | Пароль для пользователя поиска домена. |
| `Administrators group`   | Необязательно | Уровень пользователей в этой группе повышается до администраторов сайта при входе на ваше устройство. Если вы не настроите группу администраторов LDAP, до администратора сайта будет повышен уровень первой учетной записи пользователя LDAP, вошедшей на устройство. |
| `Domain base`            | Обязательно | Полное `Distinguished Name` (DN) поддерева LDAP, в котором требуется выполнить поиск пользователей и групп. Можно добавить сколько угодно имен, однако каждая группа должна быть определена в той же доменной базе, что и пользователи, которые к ней относятся. Если вы указываете ограниченные группы пользователей, то будут охвачены только пользователи, принадлежащие этим группам. Рекомендуется указать верхний уровень дерева каталогов LDAP в качестве доменной базы и использовать ограниченные группы пользователей для управления доступом. |
| `Restricted user groups` | Необязательно | Если эти группы указаны, вход будет разрешен только пользователям в этих группах. Указывать следует только общие имена (CN) групп, и можно добавить сколько угодно групп. Если группы не указаны, *все* пользователи в пределах указанной доменной базы смогут входить в ваш экземпляр {% data variables.product.prodname_ghe_server %}. |
| `User ID`                | Обязательно | Атрибут LDAP, определяющий пользователя LDAP, который пытается пройти проверку подлинности. После установки сопоставления пользователи могут изменить свои имена пользователей {% data variables.product.prodname_ghe_server %}. Для большинства установок Active Directory это поле будет иметь значение `sAMAccountName`, но для других решений LDAP, например OpenLDAP, это может быть значение `uid`. Значение по умолчанию — `uid`. |
| `Profile name`           | Необязательно | Имя, которое будет отображаться на странице профиля пользователя {% data variables.product.prodname_ghe_server %}. Если синхронизация LDAP не включена, пользователи могут изменять свои имена профилей. |
| `Emails`                 | Необязательно | Адреса электронной почты для учетной записи пользователя {% data variables.product.prodname_ghe_server %}. |
| `SSH keys`               | Необязательно | Открытые ключи SSH, подключенные к учетной записи пользователя {% data variables.product.prodname_ghe_server %}. Ключи должны быть в формате OpenSSH. |
| `GPG keys`               | Необязательно | Ключи GPG, подключенные к учетной записи пользователя {% data variables.product.prodname_ghe_server %}. |
| `Disable LDAP authentication for Git operations` | Необязательно |Если выбрать этот атрибут, для пользователей будет [отключена](#disabling-password-authentication-for-git-operations) возможность проверки подлинности операций Git с помощью паролей LDAP. |
| `Enable LDAP certificate verification` | Необязательно |Если выбрать этот атрибут, будет [включена](#enabling-ldap-certificate-verification) проверка сертификатов LDAP. |
| `Synchronization` | Необязательно |Если выбрать этот атрибут, будет [включена](#enabling-ldap-sync) синхронизация LDAP. |

### <a name="disabling-password-authentication-for-git-operations"></a>Отключение проверки паролей для операций Git

Выберите **Отключить проверку подлинности по имени пользователя и паролю для операций Git** в параметрах LDAP, чтобы обеспечить использование личных маркеров доступа или ключей SSH для доступа к Git. Это помогает предотвратить перегрузку сервера запросами проверки подлинности LDAP. Рекомендуется использовать этот параметр, так как сервер LDAP отличается довольно медленным откликом, и в сочетании с большим количеством запросов из-за опроса часто является источником проблем с производительностью и сбоев.

![Флажок отключения проверки паролей LDAP для Git](/assets/images/enterprise/management-console/ldap-disable-password-auth-for-git.png)

Если этот флажок установлен, то при попытке использования пароля для операций Git в командной строе пользователь получит следующее сообщение об ошибке: `Password authentication is not allowed for Git operations. You must use a personal access token.`

### <a name="enabling-ldap-certificate-verification"></a>Включение проверки сертификата LDAP

Выберите **Включить проверку сертификата LDAP** в параметрах LDAP, чтобы выполнялась проверка сертификата сервера LDAP, используемого с TLS.

![Флажок включения проверки сертификата LDAP](/assets/images/enterprise/management-console/ldap-enable-certificate-verification.png)

Если этот флажок установлен, выполняются следующие проверки сертификата.
- Если сертификат содержит хотя бы одно альтернативное имя субъекта (SAN), проверяется, соответствует ли одно из имен SAN имени узла LDAP. В противном случае проверяется соответствие общего имени (CN) имени узла LDAP.
- Сертификат является действительным на текущую дату.
- Сертификат подписан доверенным центром сертификации (ЦС).

### <a name="enabling-ldap-sync"></a>Включение синхронизации LDAP

{% note %}

**Примечание.** В командах, использующих синхронизацию LDAP, может быть не более 1499 членов.

{% endnote %}

Синхронизация LDAP позволяет синхронизировать пользователей и членов групп {% data variables.product.prodname_ghe_server %} с установленными группами LDAP. Это позволяет устанавливать управление доступом на основе ролей для пользователей с сервера LDAP, вместо того чтобы делать это вручную в {% data variables.product.prodname_ghe_server %}. Дополнительные сведения см. в разделе [Создание команд](/enterprise/{{ currentVersion }}/admin/guides/user-management/creating-teams#creating-teams-with-ldap-sync-enabled).

Чтобы включить синхронизацию LDAP, в параметрах LDAP выберите **Синхронизировать электронную почту**, **Синхронизировать ключи SSH** или **Синхронизировать ключи GPG**.

![Флажок синхронизации](/assets/images/enterprise/management-console/ldap-synchronize.png)

После включения синхронизации LDAP в указанный интервал времени будет запущено задание синхронизации для выполнения следующих операций с каждой учетной записью пользователя.

- Если вы разрешили встроенную проверку подлинности для пользователей за пределами вашего поставщика удостоверений, и пользователь использует встроенную проверку подлинности, задание синхронизации переходит к следующему пользователю.
- Если для пользователя не существует сопоставления LDAP, выполняется попытка сопоставить пользователя с записью LDAP в каталоге. Если пользователя не удается сопоставить с записью LDAP, задание синхронизации блокирует этого пользователя и переходит к следующему пользователю.
- Если в каталоге отсутствует и сопоставление LDAP, и соответствующая запись LDAP, этот пользователь блокируется и выполняется переход к следующему пользователю.
- Если соответствующая запись LDAP помечена как отключенная, а пользователь еще не заблокирован, этот пользователь блокируется и выполняется переход к следующему пользователю.
- Если соответствующая запись LDAP не помечена как отключенная, а пользователь заблокирован, и при этом в Центре администрирования включена _повторная активация заблокированных пользователей_, этот пользователь разблокируется.
- Если в экземпляре настроена одна или несколько ограниченных групп пользователей, а соответствующая запись LDAP не входит в одну из этих групп, пользователь блокируется.
- Если в экземпляре настроена одна или несколько ограниченных групп пользователей, соответствующая запись LDAP находится в одной из этих групп, и в Центре администрирования включена _повторная активация заблокированных пользователей_, пользователь разблокируется.
- Если соответствующая запись LDAP содержит атрибут `name`, имя профиля пользователя обновляется.
- Если соответствующая запись LDAP находится в группе "Администраторы", уровень пользователя повышается до администратора сайта.
- Если соответствующая запись LDAP отсутствует в группе "Администраторы", уровень пользователя понижается до обычной учетной записи.
- Если для электронной почты определено поле "Пользователь LDAP", параметры электронной почты пользователя синхронизируются с записью LDAP. Первая запись LDAP `mail` устанавливается в качестве основного адреса электронной почты.
- Если для открытых ключей SSH определено поле "Пользователь LDAP", открытые ключи SSH пользователя синхронизируются с записью LDAP.  
- Если для ключей GPG определено поле "Пользователь LDAP", ключи GPG пользователя синхронизируются с записью LDAP.  

{% note %}

**Примечание.** Записи LDAP можно пометить как отключенные, только если вы используете Active Directory, и атрибут `userAccountControl` присутствует и помечен как `ACCOUNTDISABLE`. Некоторые варианты Active Directory, например AD LDS и ADAM, не поддерживают атрибут `userAccountControl`.

{% endnote %}

Задание синхронизации также будет запускаться в указанный интервал времени, чтобы в каждой команде, сопоставленной с группой LDAP, выполнить следующие операции.

- Если соответствующая группа LDAP команды удалена, из команды удаляются все участники.
- Если записи членов LDAP удалены из группы LDAP, соответствующие пользователи удаляются из команды. Если пользователь больше не является членом какой-либо команды в организации, этот пользователь удаляется из организации. Если пользователь в результате теряет доступ к любым репозиториям, удаляются все частные вилки репозиториев, которые есть у этого пользователя.
- Если записи членов LDAP добавлены в группу LDAP, соответствующие пользователи добавляются в команду. Если в результате пользователь восстанавливает доступ к каким-либо репозиториям, восстанавливаются все частные вилки репозиториев, удаленные из-за потери пользователем доступа за последние 90 дней.

{% data reusables.enterprise_user_management.ldap-sync-nested-teams %}

{% warning %}

**Предупреждение системы безопасности.**

Если синхронизация LDAP включена, администраторы сайта и владельцы организации могут искать в каталоге LDAP группы для сопоставления команд.

Это может привести к раскрытию подрядчикам или другим непривилегированным пользователям конфиденциальной организационной информации, включая следующее:

- существование определенных групп LDAP, видимых *пользователю поиска домена*;
- членов группы LDAP, имеющих учетные записи пользователей {% data variables.product.prodname_ghe_server %}, которые раскрываются при создании группы, синхронизированной с этой группой LDAP.

Если раскрытие таких сведений нежелательно, ваша компания или организация должна ограничить разрешения настроенного *пользователя поиска домена* в консоли администрирования. Если такое ограничение невозможно, обратитесь в {% data variables.contact.contact_ent_support %}.

{% endwarning %}

### <a name="supported-ldap-group-object-classes"></a>Поддерживаемые классы объектов групп LDAP

{% data variables.product.prodname_ghe_server %} поддерживает следующие классы объектов групп LDAP. Группы могут быть вложенными.

- `group`
- `groupOfNames`
- `groupOfUniqueNames`
- `posixGroup`

## <a name="viewing-and-creating-ldap-users"></a>Просмотр и создание пользователей LDAP

Вы можете просматривать полный список пользователей LDAP, имеющих доступ к вашему экземпляру, и подготавливать к работе новых пользователей.

{% data reusables.enterprise_site_admin_settings.sign-in %} {% data reusables.enterprise_site_admin_settings.access-settings %}
3. В левой боковой панели выберите **Пользователи LDAP**.
![Вкладка "Пользователи LDAP"](/assets/images/enterprise/site-admin-settings/ldap-users-tab.png)
4. Чтобы найти пользователя, введите полное или частичное имя пользователя и нажмите **Поиск**. Существующие пользователи появятся в результатах поиска. Если пользователь не существует, нажмите **Создать**, чтобы подготовить к работе новую учетную запись пользователя.
![Поиск LDAP](/assets/images/enterprise/site-admin-settings/ldap-users-search.jpg)

## <a name="updating-ldap-accounts"></a>Обновление учетных записей LDAP

Пока не [включена синхронизация LDAP](#enabling-ldap-sync), автоматическая синхронизация изменений учетных записей LDAP с {% data variables.product.prodname_ghe_server %} не выполняется.

* Чтобы использовать новую группу администрирования LDAP, необходимо вручную повышать и понижать уровень пользователей в {% data variables.product.prodname_ghe_server %} для отражения изменений в LDAP.
* Для добавления или удаления учетных записей LDAP в группах администраторов LDAP следует [повышать или понижать уровни учетных записей в {% data variables.product.prodname_ghe_server %}](/enterprise/{{ currentVersion }}/admin/guides/user-management/promoting-or-demoting-a-site-administrator).
* Чтобы удалить учетные записи LDAP, [заблокируйте учетные записи {% data variables.product.prodname_ghe_server %}](/enterprise/{{ currentVersion }}/admin/guides/user-management/suspending-and-unsuspending-users).

### <a name="manually-syncing-ldap-accounts"></a>Синхронизация учетных записей LDAP вручную

{% data reusables.enterprise_site_admin_settings.sign-in %} {% data reusables.enterprise_site_admin_settings.access-settings %} {% data reusables.enterprise_site_admin_settings.search-user %} {% data reusables.enterprise_site_admin_settings.click-user %} {% data reusables.enterprise_site_admin_settings.admin-top-tab %} {% data reusables.enterprise_site_admin_settings.admin-tab %}
5. В разделе "LDAP" нажмите **Синхронизировать сейчас**, чтобы вручную обновить учетную запись с помощью данных с вашего сервера LDAP.
![Кнопка немедленной синхронизации LDAP](/assets/images/enterprise/site-admin-settings/ldap-sync-now-button.png)

Вы также можете [использовать API для активации синхронизации вручную](/enterprise/{{ currentVersion }}/user/rest/reference/enterprise-admin#ldap).

## <a name="revoking-access-to--data-variablesproductproduct_location-"></a>Отзыв доступа к {% data variables.product.product_location %}

Если [синхронизация LDAP включена](#enabling-ldap-sync), удаление учетных данных LDAP пользователя заблокирует его учетную запись после следующего запуска синхронизации.

Если синхронизация LDAP **не** включена, необходимо вручную заблокировать учетную запись {% data variables.product.prodname_ghe_server %} после удаления учетных данных LDAP. Дополнительные сведения см. в разделе [Блокировка и разблокировка пользователей](/enterprise/{{ currentVersion }}/admin/guides/user-management/suspending-and-unsuspending-users).
