FOR PRIVACY AND CODE PROTECTING REASONS THIS IS A SIMPLIFIED VERSION OF CHANGES AND NEW FEATURES

TASK DATE: 30.10.2017 - Finished: 30.10.2017

TASK LEVEL: (EASY)  

TASK SHORT DESCRIPTION: 1134 (duplicate prompt when creating a record)

GITHUB REPOSITORY CODE: fetaure/task-1134-duplicate-prompt-when-creating-a-record

CHANGES
 
	IN FILES: 
	
		network_settings_lang.php
		
			ADDED CODE: 
			
			//General Setups
			$lang['users:create:already_exists'] = 'A user with these details already exists in the database. Click CREATE to proceed.';
		
		
		
		
		members.php
		
			ADDED CODE: 
			
				/**
				 * Check new user already exists in DB or not 
				 *
				 * @input: new_member input fields' value 
				 *				member_fields for check: record_type, title, other_title, first_name, last_name, user_group(s)
				 * 
				 * @return: logical value
				*/
				public function user_already_exists()
				{
					if(!$this->input->is_ajax_request() or !$this->current_user)  {
						show_404();
					}
					
					$exploded = explode('&', $this->input->post('fields'));
					
					$fields = array();
					foreach($exploded as $key => $value) {
						$data = explode("=", $value);
						$fields[$data[0]] = $data[1];
					}
				
					..........
					
					while ($exist and list($key, $value) = each($fields)) {
						if ($value != '') {
							$where .= " AND " . $key_assists[$key] . " = '" . $value . "'";
						} 
						else {
							$exist = false;
						}
					}

					if (!$exist) {
						echo false; 
					} else {
						$this->load->model('bbusers/profile_m');
						
						$result = $this->profile_m->get_user_info(0, $where);

						echo count($result) > 0 ? true : false;
					}
				}	

				
				
				
		index.php
		
			CHANGED CODE:
			
				Added new css class to fields: new_member_field
				
				<div class="new_member alert alert-warning" style="width: 90%; margin-left: auto; margin-right: auto; display: none;">
					<?=lang('users:create:already_exists')?>
				</div>
				<fieldset>
					<ul id="addNewMemberFields">
						<li>
							<label for="f_record_type">Record Type</label>
							<div class="input">
								<?=form_dropdown('f_record_type', ci()->options->record_type_options(), null, 'class="new_member_field" id="newMemberRecordTypeTrigger"') ?>
							</div>
						</li>

						<li id="new-mem-title">
							<label for="f_title">Title</label>
							<div class="input">
								<?=form_dropdown('f_title', ci()->options->salutations(), null, 'class="new_member_field" id="newMemberTitleTrigger"') ?>
							</div>
						</li>

						<li style="display: none;" id="new-mem-other-title">
							<label for="title">Other Title</label>
							<div class="input"><?=form_dropdown('f_other_title', array('' => '-- Select an option --') + ci()->options->other_salutations(), null, 'class="new_member_field"') ?></div>
						</li>

						<li id="new-mem-first-name">
							<label for="f_first_name">First Name</label>
							<div class="input">
								<?=form_input('f_first_name', '', 'class="form-control new_member_field" style="width:236px;"') ?>
							</div>
						</li>

						<li id="new-mem-last-name">
							<label for="f_last_name">Last Name</label>
							<div class="input">
								<?=form_input('f_last_name', '', 'class="form-control new_member_field" style="width:236px;"') ?>
							</div>
						</li>

						<li id="new-mem-user-groups">
							<label for="f_newmember_group">User Groups</label>
							<div class="input">
								<?=form_chosen_multiple('f_newmember_group', $_person_type_options, null, array('extra' => 'class="form-control";'))?>
							</div>
						</li>

						<!--<li id="new-mem-organisation-groups" style="display:none">
							<label for="f_newmember_group">Organisation Groups</label>
							<div class="input">
								<//?=form_chosen_multiple('f_newmember_group', $_organisation_type_options, array('extra' => 'class="form-control";'))?>
							</div>
						</li>-->

					</ul>
				</fieldset>		

				
				
				
				

	members_administration.js
		
			CHANGED/ADDED CODE: 
			
				FROM:
				
				    $(document).on('click', "#btnAddNewMember", function () {
						$('#newMemberModal .chosen-container').each(function(i, ele){
							if ($(ele).width() == 0) {
								$(ele).css('width', '236px');
								$(ele).find('.chosen-drop').css('width', '234px');
								$(ele).find('.chosen-search input').css('width', '200px');
								$(ele).find('.chosen-field input').css('width', '225px');
							}
						})
						//Check the new user already exists in the database
						var already_exists = false;
						$('#newMemberModal').modal('show');
					});		

				TO: 			
						
				   //add a new member popup
					$(document).on('click', "#btnAddNewMember", function () {
						$('#newMemberModal .chosen-container').each(function(i, ele){
							if ($(ele).width() == 0) {
								$(ele).css('width', '236px');
								$(ele).find('.chosen-drop').css('width', '234px');
								$(ele).find('.chosen-search input').css('width', '200px');
								$(ele).find('.chosen-field input').css('width', '225px');
							}
						})
						//Check the new user already exists in the database
						var already_exists = false;
						$('#newMemberModal')
							.modal('show')			
							.find('.new_member_field, #f_newmember_group').on('change keyup', function() {
								var fields = $('.new_member_field').serialize();//+ "&f_newmember_group=" + $('#f_newmember_group').val();
								$.ajax({
									.............
								})
							});
					});	


					
					
					
		profile_m.php
		
			CHANGED CODE:
			
				FROM: 
				
					public function get_user_info($user_id) {
						$this->db->select('u.email, u.last_login, u.username, u.muted, u.active, '
										. 'u.was_offline, p.*, g.name, ppt.person_type_id, '
										. 'pt.person_type_name, pt.flexi_grp_filter_id, '
								. 'pt.read_only', FALSE)->from('users u')
								->join('profiles p', 'u.id=p.user_id')
								->join('groups g', 'g.id=u.group_id')
								->join('profiles_person_type ppt', 'p.id=ppt.row_id', 'left')
								->join('person_type pt', 'pt.id = ppt.person_type_id', 'left')
								->where('u.id', $user_id);

						$results = $this->db->get()->result();

						return $results[0];
					}

				
				TO: 
					
					public function get_user_info($user_id = 0, $where = '') {
						$sql_where = ($user_id > 0)
										? 'u.id = ' . $user_id  
										:  $where;
						$this->db->select('u.email, u.last_login, u.username, u.muted, u.active, '
										. 'u.was_offline, p.*, g.name, ppt.person_type_id, '
										. 'pt.person_type_name, pt.flexi_grp_filter_id, '
								. 'pt.read_only', FALSE)->from('users u')
								->join('profiles p', 'u.id=p.user_id')
								->join('groups g', 'g.id=u.group_id')
								->join('profiles_person_type ppt', 'p.id=ppt.row_id', 'left')
								->join('person_type pt', 'pt.id = ppt.person_type_id', 'left')
								->where($sql_where);

						$results = $this->db->get()->result();
						return count($results) > 0 ? $results[0] : array();
					}
