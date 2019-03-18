# Facebook Ads API project

The intetion of this project is to purpose an App that can create a facebook ad on behalf of a user. for this purpose some APIs have been tested but finally "facebook-php-business-sdk" as the offical sdk provided by facebook has been chosen. this was beacause of the fact that none of the other packages were as extensive and documented as original sdk.

Several steps have been identified to create an ads along with an image. these steps are as follow:

   1- Creating a Facebook app
   2- implementing App to create an ad and image via sdk
   
# 1- Creating a Facebook app
As any other app which deals with Facebook, it requires to install a Facebook App through developer portal of facebook.

it is crucial to publish this APP and request the manege_ads permission for this app from Facebook. otherwise the app wont be fully functional in last steps of creating ads.

finally an Ad account needs to be created through Ad management console of facebook.

# 2- Implementing App to create an ad and image via sdk

### installation

Although in many documentations version 3.1.* is required to be downloaded, this version is deprecated and the sdk throws error and asks for upgrading to version 3.2.*

    {
        "require": {
            "facebook/php-business-sdk": "3.2.*"
        }
    }

>Note that this version includes a bug that makes it unable to create objects as instructed in documentations. (can be discussed verbally)

### initializing API

for this purpose sdk needs APP_ID, APP_SECRET and ACCESS_TOKEN which are accessible from Facebook developer panel.

    use FacebookAds\Api;

    Api::init(env('APP_ID'), env('APP_SECRET'), env('ACCESS_TOKEN'));
    $api = Api::instance();

### initializing AdAccount

    use FacebookAds\Object\AdAccount;

    $account = new AdAccount(env('ACCOUNT_ID'));
for this step it is necessary to have an Ad Account id that we have created in last step of step 1.

all the other steps are related to this account.

### creating a campaign 
campaign saves the objectives of ads and all the ads within a campaign will have mutual objective.

    use FacebookAds\Object\Campaign;
    use FacebookAds\Object\Fields\CampaignFields;
    use FacebookAds\Object\Values\CampaignObjectiveValues;


    $campaign  = new Campaign(null, $account->id);
    $campaign->setData([
        CampaignFields::NAME => 'SandBox campaign',
        CampaignFields::OBJECTIVE => CampaignObjectiveValues::LINK_CLICKS,
    ]);

        $campaign->create([
            Campaign::STATUS_PARAM_NAME => Campaign::STATUS_PAUSED,
        ]);
    
        $campaignId = $campaign->{CampaignFields::ID};
    
        echo $campaignId;
> be aware of using correct namespaces as in many documentations they might be wrong 

### Searching Targeting
which is required to provide some attributes for ads improvemnts

        $results = TargetingSearch::search(
        $type = TargetingSearchTypes::INTEREST,
        $class = null,
        $query = 'interests',
    );

    // top result
    $target = (count($results)) ? $results->getObjects()[0] : null;
    
    // echo "Using target: ".$target->name."\n";
    
    //needs to be provided properly
    $targeting = array(
    'geo_locations' => array(
        'countries' => array('DE'),
    ),
    'interests' => array(
            array(
              'id' => $target->id,
              'name'=>$target->name,
            ),
        ),
    );
    
### creating AdSet
The AdSet holds the attributes about the "duration" of a campaign and the "budget".

AdSet as a set of Ad objects ensures all its Ad objects  an have the same targeting.


    use FacebookAds\Object\AdSet;
    use FacebookAds\Object\Fields\AdSetFields;
    use FacebookAds\Object\Values\BillingEvents;
    use FacebookAds\Object\Values\AdSetOptimizationGoalValues;
    
    $adset = new AdSet(null, $account->id);
    $adset->setData([
      AdSetFields::NAME => 'Sandbox AdSet',
      AdSetFields::CAMPAIGN_ID => $campaign->id,
      AdSetFields::DAILY_BUDGET => '100',
      AdSetFields::OPTIMIZATION_GOAL => AdSetOptimizationGoalValues::REACH,
      AdSetFields::BILLING_EVENT => BillingEvents::IMPRESSIONS,
      AdSetFields::BID_AMOUNT => 2,
      AdSetFields::TARGETING => $targeting,
      AdSetFields::START_TIME => 
        (new \DateTime("+1 week"))->format(\DateTime::ISO8601),
      AdSetFields::END_TIME => 
        (new \DateTime("+2 week"))->format(\DateTime::ISO8601),
    ]           );
    
    $adset->create();
    //echo 'AdSet  ID: '.$adset->id."\
    
### Create an AdImage
Having AdSet, now it is possible to create an Ad, however, first it is required to upload the image for the ad.

    use FacebookAds\Object\AdImage;
    use FacebookAds\Object\Fields\AdImageFields;

    $image = new AdImage(null, $account->id);
    $image->filename = 'image_path';

    $image->create();
    //echo 'Image Hash: '.$image->hash."\n";
    
this image will be attached to ad via AdCreative

### Creating an AdCreative
Format which provides layout and contains content for the ad.
creating AdCreative requires clear understanding of facebook ads rules and permissions. for eaxmple creating political ads for some of the regions requires explicit permission by the page that ads are attached to.

> note from this step Facebook app that we are using to create ads needs to be published and have manage_ads permission otherwise sdk thrws exception and won't create ads.


    use FacebookAds\Object\AdCreative;
    use FacebookAds\Object\AdCreativeLinkData;
    use FacebookAds\Object\Fields\AdCreativeLinkDataFields;
    use FacebookAds\Object\AdCreativeObjectStorySpec;
    use FacebookAds\Object\Fields\AdCreativeObjectStorySpecFields;
    use FacebookAds\Object\Fields\AdCreativeFields;
        
    $link_data = new AdCreativeLinkData(null, $account->id);
    $link_data->setData([
        AdCreativeLinkDataFields::MESSAGE => 'Sandbox AdCreative',
        AdCreativeLinkDataFields::LINK => 'FB_PAGE_URL',
        AdCreativeLinkDataFields::IMAGE_HASH => $image->hash,
    ]);

    $object_story_spec = new AdCreativeObjectStorySpec(null, $account->id);
    $object_story_spec->setData([
        AdCreativeObjectStorySpecFields::PAGE_ID => FB_PAGE_ID,
        AdCreativeObjectStorySpecFields::LINK_DATA => $link_data,
    ]);

    $creative = new AdCreative(null, $account->id);
    $creative->setData([
        AdCreativeFields::TITLE => 'Sandbox AdCreative title',
        AdCreativeFields::OBJECT_STORY_SPEC => $object_story_spec,        
    ]);
    
    $creative->create();
    //echo 'Creative ID: '.$creative->id . "\n";


### Creating an Ad

And finally we can make Ad

    use FacebookAds\Object\Ad;
    use FacebookAds\Object\Fields\AdFields;

    $ad = new Ad(null, $account->id);
    $ad->setData([
       AdFields::CREATIVE => ['creative_id' => $creative->id],
       AdFields::NAME => 'Sandbox Ad',
       AdFields::ADSET_ID => $adset->id,
    ]);
    
# Personal insights

- the focus of the project was to identify how to make and ad with an image using sdk withing limited time . yet it did not go as desired due to following reasons:
    - SDK has a bug that took some time to get around it
    - "manage_ads" permission takes a while to request and was beyond the time scope of this project
- if there is more time it is nice to create a real app that wraps sdk
- also its good to create the fully functional app with required permissions 
- despite of the points mentioned above, hard part of the project was to understand the requriements of making an Ad