drupal-webform-civicrm-relationship-save-override
=================================================

Drupal [Webform CiviCRM integration](https://drupal.org/project/webform_civicrm) module override to allow user to choose between creation of new relationship intance and updating an old one. Default version of Webform CiviCRM module allways updates existing relationship if it is found.

This modification needs to be redone for every Webform CiviCRM integration module release because it overwrites files. Currently this modification is for version `7.x-4.5`.

Modification is only needed for one file: wf_crm_webform_postprocess.inc. Main modification is in [line 873](wf_crm_webform_postprocess.inc#L873) with code `$params += $this->createNewRelationship() ? array() : $existing;` in `processRelationship()` function.

`createNewRelationship()` is new function that searches value of field `create_new_relationship`. If it's value is `1` then new relationship is allways created. Other values allow update.

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
      $params += $this->createNewRelationship() ? array() : $existing;
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
  
   /**
   * Create new relationship? Requires create_new_relationship -field to form.
   * @return boolean
   */
  private function createNewRelationship() {
    foreach($this->node->webform["components"] as $component) {
      if($component["form_key"] == "create_new_relationship") {
        $cid = $component["cid"];
        return $this->submission->data[$cid][0] == "1";
      }
    }
    return false;
  }
```

### Intallation
1. Copy wf_crm_webform_postprocess.inc to `sites\all\modules\webform_civicrm\includes` and overwrite the default version of the file.
2. Create form element with _field key_ of `create_new_relationship`. Field type should be radio buttons or select where first value is `Yes` for creating new relationship on save. First readio button value is allways `1` that translates to true in `createNewRelationship()`.
