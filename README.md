# CCaaS Skill Manager

A custom page for **Dynamics 365 Contact Center** that lets supervisors manage the routing skills of their agents from the **Copilot Service workspace**, without access to the Copilot Service admin center.

## Why

Unified routing uses skills and proficiency to decide which service representative receives which conversation. Out of the box, skills can only be maintained in the **Skills hub** and **User management** in the **Copilot Service admin center**. The Copilot Service workspace has nothing for supervisors to view or change the skills of their team.

The Skill Manager adds that to the workspace. A dedicated security role gives supervisors permission to manage skill assignments and nothing else. They get no access to the admin center, workstreams, queues or routing rules.

## Features

- Lists all active agents, with a search that matches any part of a name.
- Shows the skills and proficiency of the selected agent.
- Changes proficiency straight from the list. The change is saved immediately.
- **Add skill** dialog:
  - Defaults the rating model to the one other agents already use for that skill. You can still change it, and a note warns that routing rules depend on it.
  - Blocks duplicates: adding a skill the agent already has shows a warning.
- Removes a skill after confirmation.
- Fluent styling that matches the rest of Dynamics 365.

## How skills are stored

Unified routing stores skills in standard Universal Resource Scheduling tables. The Skill Manager uses these directly, so there are no custom tables, plug-ins or flows.

| Table | Logical name | Used for |
|---|---|---|
| Bookable Resource | `bookableresource` | The agent (one per user set up for routing) |
| Characteristic | `characteristic` | The skill |
| Bookable Resource Characteristic | `bookableresourcecharacteristic` | One row per skill assigned to an agent. The page creates, updates and deletes these rows. |
| Rating Model | `ratingmodel` | The proficiency scale |
| Rating Value | `ratingvalue` | A value on that scale, for example *Fair* or *Good* |

## Repository contents

| File | Purpose |
|---|---|
| [src/paste/0-screen-OnVisible.txt](src/paste/0-screen-OnVisible.txt) | Formula for the screen's **OnVisible** property |
| [src/paste/1-conRoot.yaml](src/paste/1-conRoot.yaml) | The page: header, agent list and skills list |
| [src/paste/2-conAddOverlay.yaml](src/paste/2-conAddOverlay.yaml) | The **Add skill** dialog |
| [src/paste/3-conRemoveOverlay.yaml](src/paste/3-conRemoveOverlay.yaml) | The **Remove skill** confirmation dialog |
| [src/scrSkillsManagement.pa.yaml](src/scrSkillsManagement.pa.yaml) | The complete screen in one file, for reference. Use the files in `src/paste` to set it up. |

## Prerequisites

- A Dynamics 365 Contact Center (or Customer Service) environment with skill-based routing set up: at least one rating model and some skills.
- Permission to create solutions, custom pages and security roles in that environment, for example System Customizer or System Administrator.

## Setup

### 1. Create a solution

1. Go to [make.powerapps.com](https://make.powerapps.com) and select your environment.
2. Select **Solutions > New solution**.
3. Name it, for example **Skill Manager**, and select your publisher.
4. Select **Create**.

### 2. Create the custom page

1. In the solution, select **New > App > Page**. Power Apps Studio opens.
2. Select **Settings > Display** and turn off **Scale to fit**.
3. Save the page as **Skill Manager**.

### 3. Add the data sources

In Power Apps Studio, select **Data > Add data** and add these Dataverse tables. Add them before pasting, or the formulas will show errors.

- Bookable Resources
- Bookable Resource Characteristics
- Characteristics
- Rating Models
- Rating Values
- Users

### 4. Paste the page

1. Open [src/paste/1-conRoot.yaml](src/paste/1-conRoot.yaml) and copy all of it. On GitHub, use the **Copy raw file** button.
2. In Power Apps Studio, open the **Tree view**, right-click the screen and select **Paste**.
3. Repeat for [2-conAddOverlay.yaml](src/paste/2-conAddOverlay.yaml), then [3-conRemoveOverlay.yaml](src/paste/3-conRemoveOverlay.yaml).
4. Check that `conAddOverlay` and `conRemoveOverlay` are listed **above** `conRoot` in the Tree view. If they aren't, right-click each one and select **Reorder > Bring to front**.
5. Select the screen, choose the **OnVisible** property and paste the contents of [0-screen-OnVisible.txt](src/paste/0-screen-OnVisible.txt).
6. Open the **App checker** and make sure there are no errors.
7. Select **Save**, then **Publish**.

### 5. Add the page to the Copilot Service workspace

1. In your solution, select **Add existing > App > Model-driven app** and add **Copilot Service workspace**.
2. Open the app in the app designer.
3. Select **Add page > Custom page**, select **Skill Manager** from the existing custom pages and select **Add**.
4. Move the page to a group where supervisors will find it, for example **Supervisor**, and set a title and icon.
5. Select **Publish**.

> Each time you publish a change to the custom page, publish the Copilot Service workspace app again. Until you do, the app keeps showing the previous version of the page.

> Copilot Service workspace is a Microsoft managed app. Your sitemap change is layered on top of it and travels with your solution. If you'd rather leave the Microsoft app untouched, add the page to a copy of the app instead.

### 6. Create the security role

1. Go to the [Power Platform admin center](https://admin.powerplatform.microsoft.com) and select **Manage > Environments**. Select your environment, then **Settings > Users + permissions > Security roles**.
2. Select **New role**, name it **Skill Manager**, select the business unit and select **Save**.
3. Set these privileges:

| Table | Create | Read | Write | Delete | Append | Append To |
|---|---|---|---|---|---|---|
| Bookable Resource Characteristic | Organization | Organization | Organization | Organization | Organization | |
| Bookable Resource | | Organization | | | | Organization |
| Characteristic | | Organization | | | | Organization |
| Rating Value | | Organization | | | | Organization |
| Rating Model | | Organization | | | | |
| User | | Organization | | | | |

4. Save the role.
5. Add it to your solution with **Add existing > Security > Security role**.
6. Assign the role to your supervisors, in addition to the roles they already use for the Copilot Service workspace, such as **Omnichannel supervisor**.

To limit supervisors to agents in their own business unit, set the Bookable Resource Characteristic privileges to **Business Unit** instead of **Organization**.

### 7. Test

- [ ] The agent list loads, and searching on part of a name works.
- [ ] Selecting an agent shows the same skills and proficiency as the admin center.
- [ ] Changing a proficiency shows a success message and is visible in the admin center.
- [ ] **Add skill** pre-selects the rating model other agents use for that skill.
- [ ] Adding a skill the agent already has shows a warning.
- [ ] Removing a skill asks for confirmation and removes it.
- [ ] A user without the **Skill Manager** role gets a permissions error when making a change.

## Troubleshooting

**A control version is rejected while pasting.** Insert that control from the **Insert** menu, check its version with **View code**, and update the version in the YAML.

**Yellow delegation warnings.** Some formulas use `FirstN()` on purpose, so that a check runs in the app instead of in Dataverse. These warnings are expected and harmless.

**Formulas that refer to choice values show errors.** Choice names can differ between environments, for example `'Status (Characteristics)'.Active`. Retype the name and pick the right one from IntelliSense.

**A dropdown shows empty names.** Classic combo boxes can get stuck after their Items formula is changed several times. Delete the control and paste it again.

## Limitations

- All active agents are listed. The page does not filter on the supervisor's own queues or team.
- The skill list includes every active characteristic, including ones the agent already has. Adding those again is blocked.
- The default rating model comes from existing assignments, not from the skill itself. If a skill is assigned with different rating models, the first one found is used.
- Changes apply to new work items. Conversations already assigned to an agent are not routed again.
- Classic controls are used, so dropdowns have square corners.

## References

- [Converge model-driven and canvas apps using the custom page](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/model-app-page-overview)
- [Add a custom page to your model-driven app](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/add-page-to-model-app)
- [Understand how the skill match algorithm works in unified routing](https://learn.microsoft.com/en-us/dynamics365/customer-service/administer/set-up-skill-based-routing)
- [Create or edit a security role](https://learn.microsoft.com/en-us/power-platform/admin/create-edit-security-role)
