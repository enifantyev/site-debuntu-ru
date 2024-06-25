2023-06-19

## Добавление REALM для подключения к LDAP-каталогу
В настройках Datacenter — Permissions — Realms добавляем новую область, где заполняем следующие поля.

<table border=1>
  <thead>
    <tr>
      <th colspan="2">General</th><th colspan="2">Sync Options</th>
    </tr>
  </thead>
  <tbody>
  <tr>
    <td>Realm</th><td>EXAMPLE.ORG</th><td>Bind User</th><td>uid=binddn_pve,cn=sysaccounts,cn=etc,dc=example,dc=org</th>
  </tr>
  <tr>
    <td>Base Domain Name</th><td>cn=accounts,dc=example,dc=org</th><td>Bind Password</th><td>***********</th>
  </tr>
  <tr>
    <td>User Attribute Name</th><td>uid</th><td>E-Mail attribute</th><td>mail</th>
  </tr>
  <tr>
    <td>Default</th><td>V</th><td>Groupname attr.</th><td>cn</th>
  </tr>
  <tr>
    <td>Server</th><td>ipa1.example.org</th><td>User classes</th><td>person</th>
  </tr>
  <tr>
    <td>Fallback Server</th><td>ipa2.example.org</th><td>Group classes</th><td>ipausergroup</th>
  </tr>
  <tr>
    <td>Port</th><td>636</th><td>User Filter</th><td>(memberOf=cn=evo_pve,cn=groups,cn=accounts,dc=example,dc=org)</th>
  </tr>
  <tr>
    <td>Verify Certificate</th><td>V</th><td>Group Filter</th><td>(cn=evo_pve_*)</th>
  </tr>
  <tr>
    <td>Required TFA</th><td>none</th><td></th><td></th>
  </tr>
  <tr>
    <td></th><td></th><td></th><td></th>
  </tr>
  <tr>
    <td></th><td></th><td></th><td></th>
  </tr>
  <tr>
    <td></th><td></th><td></th><td></th>
  </tr>
  <tr>
    <td></th><td></th><td></th><td></th>
  </tr>
</tbody>
</table>
