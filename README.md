# Live Project

## Introduction

My last two weeks at the Tech Academy were spent as part of a development team. Our task was to create a new website for a local Portland theater using an ASP.NET MVC application in C#. Much of the new website had already been created by the time of my sprint, so there many front-end tasks to be accomplished along with the continuing back-end work. This allowed me the chance to exercise my skills in HTML, CSS, JavaScript, as well as C# and SQL. Beyond raw coding experience, these two weeks exposed me to the dynamics of team development within the context of DevOps and Scrum. I also gained more fluency in the use of Visual Studio, GitHub, and other tools. I know the future will provide me many opportunities to revisit these skills and expand on them.

Below are descriptions of the stories I worked on, as well as code snippets and screenshots to better illustrate the 

## Archive Page Styling

The Archive Page, which displayed both current and past productions, had several display issued that needed to be addressed. I fixed the CSS which was improperly selecting the elements, I corrected the alternate text for each image, and I resized the Bootstrap pill badges which were large and unsightly.

## Archive Page Responsiveness

Also on the Archive Page, the production photos were not responding to changes in window sizing. If there were too many in a row, they would be squeezed thin; if there were too few, they would stretch to occupy the entire viewport. The images were contained in a card-deck, so I adjusted the minimum width to ensure they would stack before shrinking too much. I also adjusted the maximum width of each to prevent the undue stretching. Unfortunately, when I changed the card styling it adversely affected the cards on all the other pages. So I created a custom class for the archive cards and applied the changes to that one.

```
.archivecard {
    min-width: 30rem;
    max-width: 40rem;
}
```

## Easy Login Buttons

The website accomodates three types of users: Administrator, Cast Member, and Subscriber. There is a form each user must submit to be logged in properly, which requires inputs of name, user type, and password. I was tasked to create a workaround which would allow a user to bypass the login process with a click of a button. This was purely for the convenience of developers, and my code will be removed from the site before deployment.
```
<div class="col-md-4">
    <section class="loginpadding" id="loginForm">

      <h2 class="col-md-8 col-sm-8">Easy Login</h2>
      <p>This section exists for development purposes only. Please make sure that it is removed before deployment.</p>
      <hr>
      <p><input type="button" class="iconBtn" value="Sign in as an Administrator" id="AdminBtn" /></p>
      <p><input type="button" class="iconBtn" value="Sign in as a Member" id="MemberBtn" /></p>
      <p><input type="button" class="iconBtn" value="Sign in as a Subscriber" id="SubscriberBtn" /></p>

    </section>
  </div>
</div>
```
I put the JavaScript with the premade user profiles at the bottom of the page and not in a separate file. Since it won't be part of the final program, this makes it simpler to remove.
```
<script type="text/javascript">
  document.getElementById("AdminBtn").addEventListener('click', function () {
    var email = document.getElementById('Email');
    email.value = 'test@gmail.com';
    var pass = document.getElementById('Password');
    pass.value = 'Passw0rd!';
    document.getElementById("login").submit();
  });

  document.getElementById("MemberBtn").addEventListener('click', function () {
    var email = document.getElementById('Email');
    email.value = 'member.test@gmail.com';
    var pass = document.getElementById('Password');
    pass.value = 'Ih@ve12cats';
    document.getElementById("login").submit();
  });

  document.getElementById("SubscriberBtn").addEventListener('click', function () {
    var email = document.getElementById('Email');
    email.value = 'subscriber.test@gmail.com';
    var pass = document.getElementById('Password');
    pass.value = '100100St!';
    document.getElementById("login").submit();
  });
</script>
```
## CastMembers Image Update

Among the properties of the class CastMember was one called Photo. But the website also had entire class called Photo, and the designers wanted all images to be in the Photo class. My job was to replace the CastMember.Photo with a new property called PhotoId, which would store the ID of the Photo image. This story was completed in the following steps:

### Create CastMember.PhotoId to replace CastMember.Photo
The image referenced by CastMember.Photo must be referenced by a new instance of Photo, but its id must match the proper CastMember. So I decided that when instantiating a new CastMember object, I would call the CreatePhoto method and create a new Photo whose Title was identical to the CastMember's Name property. Since CreatePhoto returns the Photo.PhotoId, I assigned that value to the new CastMember.PhotoId.
```
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "CastMemberID, Name, YearJoined, MainRole, Bio, PhotoId, CurrentMember, 
        CastMemberPersonId, AssociateArtist,EnsembleMember,CastYearLeft,DebutYear")] CastMember castMember, HttpPostedFileBase file)
        {
            ModelState.Remove("CastMemberPersonID");   
            string userId = Request.Form["dbUsers"].ToString();

            if (ModelState.IsValid)
            {
                if (file != null && file.ContentLength > 0)
                {                              
                    castMember.PhotoId = PhotoController.CreatePhoto(file, castMember.Name);
                }
               
                db.CastMembers.Add(castMember);
                db.SaveChanges();
            }

            return View(castMember);
        }
```

### Update CastMember CRUD pages
All the CastMember CRUD pages now needed to be updated to accomodate the new PhotoId property. I called the DisplayPhoto method from the PhotoController on the relevant Index, Edit, Details, and Delete pages.
```
 <img class="castImage" src="@Url.Action("DisplayPhoto", "Photo", new { id = Model.PhotoId })" style="width:100%" />
```

### Display preview images
I also altered the CastMember Create and Edit pages to display a preview of the image file which the user will assign to the cast member.
```
  <div class="form-group">
    @Html.Label("Photo")
          <div class="col-md-10 formBox"      
                <img id="img" alt="" width="100" height="100" />
                <input type="file" name="file" class="fileSelect" onchange="document.getElementById('img').src = 
                    window.URL.createObjectURL(this.files[0])">
                @Html.ValidationMessageFor(model => model.PhotoId, "", new { @class = "text-danger" })
          </div>
  </div>
```
### Update seeding with CastMember.PhotoId
The website had been seeded with a dummy cast, and I had to adjust the seeding to use the new PhotoId. So I modified the Startup Page to seed all the new Photo images, then assign the CastMember.PhotoId the value of the id field in the new Photo database table.
```
    private void SeedCastPhotos()
        {
            var converter = new ImageConverter();
            // create images first
            string imagesRoot = Path.Combine(AppDomain.CurrentDomain.BaseDirectory + @"\Content\Images");

            Image image1 = Image.FromFile(Path.Combine(imagesRoot, @"London_Bauman.png"));
            Image image2 = Image.FromFile(Path.Combine(imagesRoot, @"JacQuelle_Davis.jpg"));
           
            var photos = new List<Photo>
            {
               new Photo
                {
                    OriginalHeight = image1.Height,
                    OriginalWidth = image1.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image1, typeof(byte[])),
                    Title = "London Bauman"
                },
                new Photo
                {
                    OriginalHeight = image2.Height,
                    OriginalWidth = image2.Width,
                    PhotoFile = (byte[])converter.ConvertTo(image2, typeof(byte[])),
                    Title = "Jacquelle Davis"
                }
            };
            photos.ForEach(Photo => context.Photo.AddOrUpdate(p => p.PhotoFile, Photo));
            context.SaveChanges();
        }

        //Seeding database with dummy CastMembers
        private void SeedCastMembers()
        {
            //Add photos of cast members
            //string imagesRoot = Path.Combine(AppDomain.CurrentDomain.BaseDirectory + @"\Content\Images");

            var castMembers = new List<CastMember>
            {
                new CastMember{Name = "London Bauman", MainRole = Enum.PositionEnum.Actor,
                Bio = "London Bauman is an actor, writer, sound designer, and Theatre Vertigo company member. " +
                "As an artist, he is interested in immersive physical theatre, magical realism, and new collaborative works." +
                " Selected work includes the role of Torch in Beirut (The Steep & Thorny Way to Heaven), " +
                "Barnaby the Barkeep in the devised Western Melodrama Bang! (Action/Adventure’s pilot season) " +
                "and sound design / original compositions for The Romeo and Juliet Project (Enso Theatre Ensemble). " +
                "In August, London will be traveling to the Edinburgh Fringe Festival in Scotland as Robert in Chet Wilson’s new play
                Gayface.",
                
                PhotoId = context.Photo.Where(photo => photo.Title == "London Bauman").FirstOrDefault().PhotoId, 
                CurrentMember = true,},

                new CastMember{Name = "Jacquelle Davis", MainRole = Enum.PositionEnum.Actor,
                Bio = "Jacquelle Davis is a proud Portland native and member of Theatre Vertigo. " +
                "She studied acting at Willamette University. Jacquelle performs regularly with her beloved improv group, " +
                "No Filter. Her favorite roles include Jane Fonda in That Pretty Pretty; " +
                "Or, The Rape Play, and Box Worker 2 in Box. Jacquelle loves puns and pickles..",
                
                PhotoId = context.Photo.Where(photo => photo.Title == "Jacquelle Davis").FirstOrDefault().PhotoId,
                CurrentMember = true, }
                
                };

            castMembers.ForEach(castMember => context.CastMembers.AddOrUpdate(c => c.Name, castMember));
            context.SaveChanges();
        }
```

