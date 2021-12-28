# http-svn.code.sf.net-p-docutils-code-trunk
#! /bin/bash # $Id: docutils-update.local 8792 2021-07-07 11:52:58Z grubert $ # # สคริปต์นี้อัปเดตเว็บไซต์ Docutils # # เว็บรูทประกอบด้วย  # # * ไฟล์และไดเรกทอรีจาก ``trunk/web``: #    # * ไฟล์และไดเรกทอรีจาก ``trunk/docutils``: # ไฟล์ทั้งหมดเพื่อให้ง่ายต่อการอ้างอิงในอีเมล # # * และ ``ลำตัว/แซนด์บ็อกซ์`` # # ความสนใจ # # เอกสาร .html ใดๆ ที่มีไฟล์ .txt ที่เกี่ยวข้องจะถูกสร้างขึ้นใหม่  # หาก .txt มีการเปลี่ยนแปลง แต่ไม่มีการสร้างไฟล์ .html ใหม่ # # * เรื่องตลก: sf ซ่อนไฟล์ README.txt # # ความสนใจ # # ไดเร็กทอรีอาจมีไฟล์ Makefile.docutils-update ด้วย #คำแนะนำพิเศษ. ใช้ในเอกสาร/ผู้ใช้เพื่อโทร rst2s5 # เพิ่ม --smartquotes=true เพื่อแปลง smartquotes # อาจเพิ่มการรักษาพิเศษและลบวิธีแก้ปัญหาทั่วไปนี้ # # ตัวเลือก: # -f อย่าให้ข้อเสนอแนะ # -t เรียกใช้สคริปต์ในโหมดติดตาม ("set -o xtrace") # -u สร้าง .html ใหม่โดยไม่มีเงื่อนไข # -v เรียกใช้อย่างละเอียด # -q วิ่งเงียบ # # ข้อกำหนดเบื้องต้น:  #  # ออกเมื่อผิดพลาด ตั้ง -e  # ทำให้กลุ่มไฟล์ที่สร้างขึ้นใหม่ทั้งหมดเขียนได้ umask 002  # URL สำหรับการชำระเงินโครงการ SVN: svnurl=http://svn.code.sf.net/p/docutils/code/trunk  htmlfilelist=`pwd`/htmlfiles.lst basedir=`pwd`/update-dir ถ้า [ ! -e $basedir ] ; แล้ว     ทดสอบ -d $basedir || mkdir $basedir fi โครงการ=docutils # $auxdir ไม่ใช่สาธารณะ ... และไม่จำเป็นหากเรียกใช้ในเครื่อง  auxdir=$basedir/aux ทดสอบ -d $auxdir || mkdir $auxdir # $htdocsdest เป็นปลายทางสำหรับ htdocs และจะถูกย้ายไปที่ # เซิร์ฟเวอร์อื่นในภายหลัง; ดังนั้นเราจึงทำให้มันไม่เป็นสาธารณะ (ภายใต้ $auxdir) htdocsdest=$auxdir/htdocs ทดสอบ -d $htdocsdest || mkdir $htdocsdest # จะสร้างสแน็ปช็อตได้ที่ไหน (ไม่ใช่สาธารณะ) snapshotdir=$auxdir/snapshots ทดสอบ -d $snapshotdir || mkdir $snapshotdir  # htdocs ไดเรกทอรีบน SF.net remoteproject=/home/project-web/docutils remotehtdocs=$remoteproject/htdocs  # ชำระเงินในพื้นที่ pylib=$auxdir/lib/python lib=$pylib/$project # ล็อคไดเรกทอรี lockdir=$auxdir/lock  # URL ฐานโครงการ (สำหรับแผนผังเว็บไซต์) โดยไม่มีเครื่องหมายทับ # TODO เปลี่ยนเป็น .io ? baseurl="http://docutils.sourceforge.net"  ส่งออก PYTHONPATH=$pylib:$lib:$lib/extras ส่งออก PATH=$lib/tools:$PATH  ติดตาม=0 ไม่มีเงื่อนไข=0 ละเอียด=0 ข้อเสนอแนะ=1  ในขณะที่ getopts ftuv opt ทำ     กรณี $ เลือกใน         ฉ) ข้อเสนอแนะ=;;         t) ติดตาม=1;;         u) ไม่มีเงื่อนไข=1;;         v) ละเอียด=1;;         q) ละเอียด=0;;         \?) ทางออก 2;;     esac เสร็จแล้ว shift `expr $OPTIND - 1`  ฟังก์ชัน print_feedback () {     ทดสอบ $feedback &amp;&amp; echo "$1" || จริง }  print_feedback 'กำลังเริ่มการรันการอัปเดตเอกสาร...'  ถ้า [ $trace -eq 1 -o $verbose -eq 1 ] ; แล้ว     set -o xtrace fi  #รับล็อคอิน ถ้า ! mkdir $lockdir; แล้ว     เสียงก้อง     echo ไม่สามารถสร้างล็อกไดเร็กทอรีที่     echo $lockdir     เสียงก้อง     echo โปรดตรวจสอบให้แน่ใจว่าไม่มีผู้ใช้รายอื่นกำลังเรียกใช้สคริปต์นี้     echo และลบไดเร็กทอรี     ทางออก 1 fi # ทำความสะอาดเสมอเมื่อออก กับดัก "rm -rf $lockdir; trap - 0; exit 1" 0 1 2 3 15 # ตรวจสอบให้แน่ใจว่าไดเร็กทอรีล็อกถูกลบ (เช่น rwx) โดย group . อื่น # สมาชิก (ในกรณีที่สคริปต์นี้ขัดข้องหลังจากคัดลอกไฟล์ลงใน # ไดเรกทอรี) chmod 0770 $lockdir  #อัพเดทพื้นที่ห้องสมุด ถ้า [ -e $lib ] ; แล้ว     cd $lib     svn ขึ้น --เงียบ อื่น     ทดสอบ -d $pylib || mkdir -p $pylib     cd $pylib     svn ชำระเงิน $svnurl/docutils fi  # -------------------- ภาพรวม: --------------------  #รวบรวมวัตถุดิบ cd $snapshotdir สำหรับ DIR ในเว็บ docutils sandbox ; ทำ     ทดสอบ -d $DIR || svn ชำระเงิน $svnurl/$DIR เสร็จแล้ว # BUG หากเช็คเอาท์ครั้งแรก มีการเปลี่ยนแปลง haschanges="`svn up docutils sandbox web | grep -v '^At revision '; true`"  # ตรวจสอบให้แน่ใจว่าได้ตั้งค่าการอนุญาตไดเรกทอรีที่เหมาะสมเพื่อให้ไฟล์สามารถ # แก้ไขโดยผู้ใช้หลายคน การเปลี่ยนการอนุญาตของไฟล์คือ # อาจไม่จำเป็นเพราะไฟล์สามารถลบและสร้างใหม่ได้ # อย่าเปลี่ยนการอนุญาตของไดเร็กทอรี aux เพื่อให้ไม่เป็นสาธารณะ # (แต่เปลี่ยนการอนุญาตสำหรับไดเรกทอรีย่อยทั้งหมด) ค้นหา $basedir -name aux -o -type d -print0 | xargs -0 chmod ug+rwxs 2> /dev/null || จริง  # สร้างสแน็ปช็อต ไม่รวม='--ไม่รวม=.svn' tar -cz $exclude -f $project-snapshot.tgz $project tar -cz $exclude -f $project-sandbox-snapshot.tgz sandbox  # -------------------- htdocs: --------------------  cd $snapshotdir  # TODO นี้ใช้ไม่ได้กับ macosx ฟังก์ชัน copy_to_htdocsdest () {     ค้นหา "$@" -type d -name .svn -prune -o \( -type f -o -type l \) -print0 | \         xargs -0 cp --no-dereference --update --parents \             --target-directory=$htdocsdest }  # อัปเดต htdocs copy_to_htdocsdest sandbox (cd $project; copy_to_htdocsdest *) (เว็บซีดี copy_to_htdocsdest * .[^.]*)  # อัปเดตเอกสาร HTML cd $htdocsdest/tools  ถ้า [ $trace -eq 0] ; แล้ว     ตั้งค่า +o xtrace fi  # 1 Makefiles ท้องถิ่น สำหรับ makefile ใน `find .. -name Makefile.docutils-update` ; ทำ     dir=`dirname $makefile`     ( cd $dir ; make -f Makefile.docutils-update -s ) เสร็จแล้ว  cd $htdocsdest  # 2 สร้างไฟล์ html ที่ว่างเปล่าและเก่าเพื่อบังคับให้สร้าง # สำหรับ txt ใด ๆ ภายใต้ เอกสาร ? ค้นหาเอกสาร -type f -and -name \*.txt -print | ( \ ในขณะที่อ่าน -r txtfile ; ทำ     dir=`dirname $txtfile`     base=`ชื่อฐาน $txtfile .txt`     htmlfile=$dir/$base.html     ถ้า [ ! -e $htmlfile ] ; แล้ว         print_feedback "แตะ $htmlfile"         แตะ -t 200001010101 $htmlfile     fi เสร็จแล้ว )  # สำหรับ README.txt ใด ๆ ภายใต้แซนด์บ็อกซ์ ค้นหา sandbox -type f -and \( -name README.txt -o -name README \) -print | ( \ ในขณะที่อ่าน -r txtfile ; ทำ     dir=`dirname $txtfile`     base=`ชื่อฐาน $txtfile .txt`     htmlfile=$dir/$base.html     ถ้า [ ! -e $htmlfile ] ; แล้ว         print_feedback "แตะ $htmlfile"         แตะ -t 200001010101 $htmlfile     fi เสร็จแล้ว )  # สำหรับไฟล์ใด ๆ ใน htmlfilelist ให้สร้างไฟล์ที่หายไปด้วย mtime แบบเก่า ในขณะที่อ่าน -r htmlfile ; ทำ     ถ้า [ ! -d `dirname $htmlfile` ] ; แล้ว         print_feedback "รายการไฟล์ html แบบเก่า: $htmlfile"     เอลฟ์ [ ! -e $htmlfile ] ; แล้ว         print_feedback "แตะ $htmlfile"         แตะ -t 200001010101 $htmlfile     fi เสร็จสิ้น &lt; $htmlfilelist  # 3. สร้าง / สร้าง html จาก txt . อีกครั้ง cd $htdocsdest  # สิ่งที่ต้องทำ buildhtml.py ?  สำหรับ htmlfile ใน `find . -name '*.html'' ; ทำ     dir=`dirname $htmlfile`     base=`ชื่อฐาน $htmlfile .html`     # การทดสอบการใช้งานและแซนด์บ็อกซ์อาจไม่มี reST และ html ในไดเรกทอรีเดียวกัน     ถ้า [ "$base" == "standalone_rst_html4strict" ] ; แล้ว         print_feedback "ข้าม: $dir $base"     อื่น         txtfile=$dir/$base.txt         ถ้า [ ! -e $txtfile ] ; แล้ว             txtfile=$dir/$base.rst         fi         ถ้า [ ! -e $txtfile ] ; แล้ว             txtfile=$dir/$base         fi         ถ้า [ ! -e $txtfile ] ; แล้ว             print_feedback "เตือน: ไม่พบอินพุต: $dir $base"         อื่น             ถ้า [ $unconditional -eq 1 -o $txtfile -nt $htmlfile ] ; แล้ว                 ถ้า [ "${base:0:4}" == "pep-" ] ; แล้ว                     print_feedback "$txtfile (PEP)"                     หลาม $lib/tools/rstpep2html.py --config=$dir/docutils.conf $txtfile $htmlfile                     haschanges=1                 อื่น                     print_feedback "$txtfile"                     หลาม $lib/tools/rst2html5.py --config=$dir/docutils.conf $txtfile $htmlfile                     haschanges=1                 fi             fi         fi     fi เสร็จแล้ว  ถ้า [ $trace -eq 1 -o $verbose -eq 1 ] ; แล้ว     set -o xtrace fi  # -------------------- แผนผังเว็บไซต์ XML สำหรับเครื่องมือค้นหา: --------------------  cd $htdocsdest  # อัปเดตแผนผังเว็บไซต์เฉพาะในกรณีที่มีการเปลี่ยนแปลงเนื่องจากต้องใช้เวลา # เวลา CPU มาก ถ้า test -n "$ haschanges"; แล้ว     (         echo '&lt;?xml version="1.0" encoding="UTF-8"?>'         echo '&lt;urlset xmlns="http://www.google.com/schema/sitemap/0.84">'         ถ้า [ $trace -eq 0] ; แล้ว             ตั้งค่า +o xtrace         fi         หา . -name '.[^.]*' -prune -o -type d -printf '%p/\n' \                 -o \( -type f -o -type l \) -print | \             ในขณะที่อ่านฉัน; ทำ                 #i คือชื่อไฟล์                 ถ้าทดสอบ "$i" == ./; แล้ว                     #โฮมเพจ.                     i=index.html                     url="$baseurl/"                 elif ทดสอบ "$i" == ./sitemap -o "${i: -1}" == / -a -f "${i}index.html"; แล้ว                     # นี่คือไดเร็กทอรีและมี index.html ดังนั้นเราจึง                     #ไม่ต้องใส่ก็ได้                     ดำเนินต่อ                 อื่น                     url="$baseurl${i:1}"                     url="${url// /%20}"                 fi                 lastmod="`date --iso-8601=วินาที -u -r "$i"`"                 # Google ต้องการเครื่องหมายทวิภาคหน้าสองหลักสุดท้าย                 lastmod="${lastmod::22}:00"                 ถ้าทดสอบ "${i: -5}" == .html; แล้ว                     # ไฟล์ HTML (รวมถึงโฮมเพจ) มีลำดับความสำคัญสูงสุด                     ลำดับความสำคัญ=1.0                 elif ทดสอบ "${i: -4}" == .txt; แล้ว                     # ไฟล์ข้อความมีลำดับความสำคัญปานกลาง                     ลำดับความสำคัญ=0.5                 อื่น                     # อย่างอื่น (ไฟล์ต้นฉบับ ฯลฯ) มีลำดับความสำคัญต่ำ                     ลำดับความสำคัญ=0.2                 fi                 echo "&lt;url>&lt;loc>$url&lt;/loc>&lt;lastmod>$lastmod&lt;/lastmod>&lt;priority>$priority&lt;/priority>&lt;/url>"             เสร็จแล้ว         ถ้า [ $trace -eq 1 -o $verbose -eq 1 ] ; แล้ว             set -o xtrace         fi         เสียงสะท้อน '&lt;/urlset>'     ) > แผนผังเว็บไซต์     gzip -f แผนผังเว็บไซต์ fi  # -------------------- ผลักดันการเปลี่ยนแปลงไปยังเซิร์ฟเวอร์ระยะไกล --------------------  # sourceforge ไม่อนุญาตให้เข้าถึงเชลล์อีกต่อไป ใช้ rsync ผ่าน ssh # ระบุผู้ใช้ของคุณใน .ssh/config  cd $htdocsdest  print_feedback "rsync ถึง sf" # ห้ามใช้ -a เพื่อหลีกเลี่ยง "ไม่สามารถตั้งค่าการอนุญาต" # -t รักษาเวลาในการแก้ไข แต่การเช็คเอาต์ svn ใหม่มี modtime ใหม่ rsync -e ssh -r -t ./ web.sourceforge.net:$remotehtdocs  กับดัก - 0 1 2 3 15 rm -rf $lockdir print_feedback '...docutils-update done'  # ตัวแปรท้องถิ่น: # เยื้องแท็บโหมด: ไม่มี # จบ:
