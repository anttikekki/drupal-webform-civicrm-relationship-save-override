drupal-webform-civicrm-relationship-save-override
=================================================

Drupal [Webform CiviCRM integration](https://drupal.org/project/webform_civicrm) module override to force creation of new relationship intance on every save. Default version of Webform CiviCRM module updates existing relationship if it is found.

This modification needs to be redone for every Webform CiviCRM integration module release. Currently this modification is for version `7.x-4.5`.

Modification is only neede for one file and the only modification to wf_crm_webform_postprocess.inc is commenting out line 873 with code `$params += $existing;` in `processRelationship()` function:

```php
private function processRelationship($params, $cid1, $cid2) {
    if (!empty($params['relationship_type_id']) && $cid2 && $cid1 != $cid2) {
      list($type, $side) = explode('_', $params['relationship_type_id']);
      $existing = $this->getRelationship(array($params['relationship_type_id']), $cid1, $cid2);
      $perm = wf_crm_aval($params, 'relationship_permission');
      // Swap contacts if this is an inverse relationship
      if ($side == 'b' || ($existing && $existing['contact_id_a'] != $cid1)) {
        list($cid1, $cid2) = array($cid2, $cid1);
        if ($perm == 1 || $perm == 2) {
          $perm = $perm == 1 ? 2 : 1;
        }
      }
      //$params += $existing; //Force creation of new relationship intance on every save
      $params['contact_id_a'] = $cid1;
      $params['contact_id_b'] = $cid2;
      $params['relationship_type_id'] = $type;
      if ($perm) {
        $params['is_permission_a_b'] = $params['is_permission_b_a'] = $perm == 3 ? 1 : 0;
        if ($perm == 1 || $perm == 2) {
          $params['is_permission_' . ($perm == 1 ? 'a_b' : 'b_a')] = 1;
        }
      }
      unset($params['relationship_permission']);
      wf_civicrm_api('relationship', 'create', $params);
    }
  }
```

### Intallation
Copy wf_crm_webform_postprocess.inc to `sites\all\modules\webform_civicrm\includes` and overwrite the default version of the file.
