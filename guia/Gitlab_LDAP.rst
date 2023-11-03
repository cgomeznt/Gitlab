GitLab LDAP
====================

Editar el siguiente archivo "/etc/gitlab/gitlab.rb" y configurar los campos::


	# vi /etc/gitlab/gitlab.rb
	
	[...]

	### GitLab Mattermost
	###! These settings are void if Mattermost is installed on the same omnibus
	###! install
	# gitlab_rails['mattermost_host'] = "https://mattermost.example.com"

	### LDAP Settings
	###! Docs: https://docs.gitlab.com/omnibus/settings/ldap.html
	###! **Be careful not to break the indentation in the ldap_servers block. It is
	###!   in yaml format and the spaces must be retained. Using tabs will not work.**

	gitlab_rails['ldap_enabled'] = true
	gitlab_rails['ldap_servers'] = {
	'main' => {
	  'label' => 'Active Directory Credicard',
	  'host' =>  '10.132.0.3',
	  'port' => 389,
	  'uid' => 'sAMAccountName',
	  'verify_certificates' => false,
	  'bind_dn' => 'CN=usrfaq,OU=Cuentas de Servicios,OU=VicePresidencia de Infraestructura,OU=OU Credicard Caracas,DC=credicard,DC=com,DC=ve',
	  'password' => 'vY1%mHWKny',
	  'active_directory' => true,
	  'base' => 'OU=OU Credicard Caracas,DC=credicard,DC=com,DC=ve',
	  'group_base' => 'OU=OU Credicard Caracas,DC=credicard,DC=com,DC=ve',
	  'admin_group' => 'Global Admins'
	  }
	}

	[...]
	
Luego aplicamos el reconfigure::

	# gitlab-ctl reconfigure
